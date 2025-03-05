
- 고객문의응대 채팅
- 고객은 어드민에게 하나의 채팅방을 생성해 문의할 수 있다.
	- 사용자는 나의 방에 항상 들어가 있다.
- 어드민은 모든 고객의 채팅방을 확인하고 답변할 수 있다.
	- 어드민은 모든 사용자의 방에 들어갈 수 있다.
- 로그인한 유저만 채팅을 이용할 수 있어야 한다.
	- 유저 검증기능 추가




### socket 연결이 되었을 때, 연결이 끊어졌을 때 구현

### 서비스단에서 연결된 소켓을 기억하게 만든다.
- Map으로 소켓을 저장하는 공간 connectedClients을 만든다.
	- Map 객체는 키-값 쌍인 집합이다.
	- Map 객체는 오직 하나만 존재한다.
	- 추후에 redis등을 이용할 수 있다.
-  connectedClients에 소켓을 저장하고 제거할 수 있는 함수를 만든다.
	- Map 객체는 set(key, value), delete(key)로 데이터를 저장할 수 있다.
```node
	@Injectable()  
	export class ChatService {  
	//<userId, Socket>
	  private readonly connectedClients = new Map<number, Socket>();

	  registerClient(userId: number, client:Socket){
		 this.connectedClients.set(userId, client)
	  }

	  removeClient(userId:number){
		 this.connectedClients.delete(userId)
	  }	
	}
```

### gateway에서 해당 서비스를 구현하는 메소드를 구현다.
- OnGatewayConnection, OnGatewayDisconnect 인터페이스를 구현한다.
	- OnGatewayConnection은 handleConnection메소드를 가지고 있다.
		- 로그인 유저의 토큰을 받아 검증한 다음 parsing하여 payload를 가져온다.
		- payload가 있다면 socket에 payload를 담은 다음 registerClient를 생성한다.
		- 없다면 socket 연결을 종료한다.
	- OnGatewayDisconnect은 handleDisconnect메소드를 가지고 있다.
		- 다음 메소드를 실행하려면 로그인 검증이 되어야만 한다. socket에 유저정보를 가지고 있는지 확인한다.
		- socket에 유저정보가 있다면 removeClient를 실행한다.

### 방에 들어가는 메소드를 구현한다.
- userId와 socket을 이용해 chatRoom 테이블에 저장된 내 chat들을 불러온다.
	- chatRoom, chat, user 엔티티를 이용한다.
	- 일반유저라면 한개의 chatRoom id를 가지고 있을 것이고 어드민이라면 여러개의 chatRoom id를 가지고 있을 것이다. 이 Id로 socket에 접속한다.

```node
@Injectable()  
	export class ChatService {  
	//<userId, Socket>
	  private readonly connectedClients = new Map<number, Socket>();

	  registerClient(userId: number, client:Socket){
		 this.connectedClients.set(userId, client)
	  }

	  removeClient(userId:number){
		 this.connectedClients.delete(userId)
	  }	
	
	async joinUserRooms(user: { sub: number }, client: Socket) {  
	  const chatRooms = await this.chatroomRepository  
		.createQueryBuilder('chatRoom')  
		.innerJoin('chatRoom.users', 'user', 'user.id = :userId', {  
		  userId: user.sub,  
		})  
		.getMany();  

		chatRooms.forEach((chatRoom) => {  
			// 여러 종류의 채팅방을 생성해야 할 경우
			// client.join(`chatRoom/${chatRoom.id.toString()}`);
			client.join(chatRoom.id.toString());  
		});  
	}
}

```

- joinUserRooms이 실행되는 시점은 handleConnection 시점이다. Map에 userId, Socket을 저장한 다음에 실행한다.
```node
async handleConnection(client: Socket) {  
  try {  
    // Bearer $token  
    const rawToken = client.handshake.headers.authorization;  
    const payload = await this.authService.parseBearerToken(rawToken, false);  
    if (payload) {  
      client.data.user = payload;  
      this.chatService.registerClient(payload.sub, client);  
      await this.chatService.joinUserRooms(payload, client);  
    } else {  
      client.disconnect();  
    }  
  } catch (error) {  
    console.log(error);  
    client.disconnect();  
  }  
}
```

### 유저가 메시지를 생성하는 메소드를 구현한다.
- 메시지를 작성하면 postgres에 chatroom, chat에 데이터가 저장된다.
- 둘 이상의 데이터를 저장하기 위해 transaction interceptor를 생성한다.
	- Socket(client)정보를 불러오기 위해 실행 컨텍스트에서 websocket 정보를 불러온다.
	- 불러온 client에 QueryRunner를 담고 transaction 로직을 생성한다.

```node
@Injectable()  
export class WsTransactionInterceptor implements NestInterceptor {  
  constructor(private readonly dataSource: DataSource) {}  
  
  async intercept(  
    context: ExecutionContext,  
    next: CallHandler<any>,  
  ): Promise<Observable<any>> {  
    const client = context.switchToWs().getClient();  
  
    const qr = this.dataSource.createQueryRunner();  
  
    await qr.connect();  
    await qr.startTransaction();  
  
    client.data.queryRunner = qr;  
  
    return next.handle().pipe(  
      catchError(async (err) => {  
        await qr.rollbackTransaction();  
        await qr.release();  
  
        throw err;  
      }),  
      tap(async () => {  
        await qr.commitTransaction();  
        await qr.release();  
      }),  
    );  
  }  
}
```

- interceptor에 담을 QueryRunner를 불러오기 위한 데코레이터를 생성한다.
	- 실행 컨텍스트에서 websocket에 담은 QueryRunner 정보를 불러올 수 있다.
```node
export const QueryRunner = createParamDecorator(
(data: any, context: ExecutionContext) => {  
  const client = context.switchToWs().getClient();  
  
  if (!client || !client.data || !client.data.queryRunner) {  
    throw new InternalServerErrorException(  
      'Query Runner 객체를 찾을 수 없습니다.',  
    );  
  }  
  return client.data.queryRunner;  
},
)
```

- gateway에서 메시지 생성 메소드를 만들고 그곳에 interceptor와 decorator를 추가한다.
	- CreateChatDto는 messag와 room정보를 가지고 있다.
```node
@SubscribeMessage('sendMessage')  
@UseInterceptors(WsTransactionInterceptor)  
async handleMessage(  
  @MessageBody() body: CreateChatDto,  
  @ConnectedSocket() client: Socket,  
  @WsQueryRunner() qr: QueryRunner,  
) {  
  const payload = client.data.user;  
  await this.chatService.createMessage(payload, body, qr);  
}
```


### 채팅방을 생성하거나 생성한 채팅방에 메시지를 보낸다.
- 로그인 유저 정보 확인
	- 관리자는 모든 채팅방 정보를 가져온다.
		- 채팅방이 없다면 빈 리스트를 가져올 것이다.
	- 일반유저는 자신의 채팅방 내용을 가져온다.
		- 채팅방이 없다면 새로 생성한다.

```node
```