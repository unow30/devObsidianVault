[[Websocket Soket Io|이론 정리]] 
- 관련 기능 설치
	`pnpm i @nestjs/websockets, @nestjs/websockets, socket.io, @nestjs/platform-socket.io`
- socket.io 폴더 파일 생성(채팅 파일)
	`nest g chat`
	`what transport layer do you use?: websocket`

### 예제코드
```node.js
import {  
  WebSocketGateway,  
  SubscribeMessage,  
  MessageBody,  
  ConnectedSocket,  
} from '@nestjs/websockets';  
import { ChatService } from './chat.service';  
import { Socket } from 'socket.io';  
  
@WebSocketGateway()  
export class ChatGateway {  
  constructor(private readonly chatService: ChatService) {}  

  //client to server: 서버가 메시지를 받는다.
  @SubscribeMessage('receiveMessage')  
  async receiveMessage(  
    @MessageBody() data: { message: string },  
    @ConnectedSocket() client: Socket,  
  ) {  
    console.log('receiveMessage', data, client);  
  } 

  // server to client: 서버가 메시지를 보낸다.
  @SubscribeMessage('sendMessage')  
  async sendMessage(  
    @MessageBody() data: { message: string },  
    @ConnectedSocket() client: Socket,  
  ) {  
    client.emit('sendMessage', { ...data, from: 'server' });  
  }  
}

```

	- nestjs 실행 로그에 gateway 뜬다.
```
[Nest] 5081  - 03/02/2025, 1:55:37 PM     LOG [WebSocketsController] ChatGateway subscribed to the "receiveMessage" message +20ms

또는 winston 로그
2025-03-02T04:57:58.580Z WebSocketsController info ChatGateway subscribed to the "receiveMessage" message
```


### 웹소켓 연결 모습
![[Pasted image 20250302140502.png|500]]
### client to server(receiveMessage)
- receiveMessage 이벤트로 메시지 전송
	- receiveMessage
	- 메시지를 보내면 서버로 보내면↑ 서버는 받은 메시지를 콘솔로 확인할 수 있다.
	  ![[Pasted image 20250302140750.png|500]]
	![[Pasted image 20250302142902.png|500]]
### server to client(sendMessage)
- 이벤트 연결
	- sendMessage 이벤트를 postman(client)가 listen하도록 만든다.
	- 서버가 보내줄 요청을 Message에 이벤트명과 함께 적어서 보낸다.
		![[Pasted image 20250302141634.png|500]]
	- sendMessage 이벤트로 서버에서 보낸 결과
		1. postman에서 서버가 보낼 데이터인 message: hello ws를 보낸다.↑
		2. 서버는 해당 데이터를 받고 가공한 다음 보내준다. (from: server 추가) ↓
			![[Pasted image 20250302141729.png|500]]
	