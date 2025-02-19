## 토큰 캐싱하기
- 토큰은 생성 후 exp 시간동안 유효하다.
- 검증을 받은 토큰은 미들웨어에서 캐싱한다면 매 요청에서 토큰 유효성 검사를 건너뛸 수 있다.
- 토큰 유효시간을 고려하여 캐싱시간을 지정한다.

```node
export class BearerTokenMiddleware implements NestMiddleware {  
  constructor(  
    private readonly jwtService: JwtService,  
    private readonly configService: ConfigService,  
    @Inject(CACHE_MANAGER)  
    private readonly cacheManager: Cache,  
  ) {}
	...
	//유효성 검증한 토큰
	const tokenKey = `TOKEN_${token}`;  

	//저장한 캐싱값이 있다면 바로 불러온다. 그는 
	const cachedPayload = await this.cacheManager.get(tokenKey);    
	if (cachedPayload) {  
	  req['user'] = cachedPayload;  
	  return next();  
	}
	
	const payload = '토큰 유효성 검증 및 디코딩 완료값'
	
	//저장할 캐시 키 지정
	const tokenKey = `TOKEN_${token}`;
	
	//payload 캐싱
	const expiryDate = +new Date(payload['exp'] * 1000);  
	const now = +Date.now();  
	const diffInSeconds = (expiryDate - now) / 1000;  
	await this.cacheManager.set(  
	  tokenKey,  
	  payload,
	  
	  //계산시간 고려 토큰 만료시간에서 30초 뺀다. 최소 1ms 지속한다.;
	  Math.max((diffInSeconds - 30) * 1000, 1),
	)
	
	req['user'] = payload
	next()
}
```

## 토큰 블록하기
- 특정 토큰을 확인하여 요청을 못하게 만든다.
	- 새로운 기기로 로그인 시 확인절차, 신규 로그인 기기의 접근제한
	- 이상한 요청을 하는 토큰을 확인시 다른 요청을 못하도록 블록
- 여기선 블락된 토큰은 캐시에 저장한다.
	- 토큰을 블락하는 api를 생성한다.
	- 미들웨어에서 블락한 토큰을 검증한다.
	- 캐싱 유효기간 동안 해당 토큰은 다른 요청을 못하도록 막는다.

```node
//컨트롤러
@Post('token/block')  
blockToken(@Body('token') token: string) {  
  return this.authService.tokenBlock(token);  
}

//서비스: 토큰을 캐싱으로 저장한다.
@Injectable()  
export class AuthService {  
  constructor(  
    @InjectRepository(User)  
    private readonly userRepository: Repository<User>,  
    private readonly configService: ConfigService,  
    private readonly jwtService: JwtService,  
    @Inject(CACHE_MANAGER)  
    private readonly cacheManager: Cache,  
  ) {}  
  
async tokenBlock(token: string) {  
    //유효한 토큰이라 가정하므로 디코드만 한다.
    const payload = this.jwtService.decode(token);  
    //캐싱의 키 지정
    const tokenKey = `BLOCKED_TOKEN_${token}`;  
	
	//블락 유효시간 지정
    const expiryDate = +new Date(payload['exp'] * 1000);  
    const now = +Date.now();  
    const diffInSeconds = (expiryDate - now) / 1000;  
    console.table({  
      expiryDate: expiryDate,  
      now: now,  
      diffInSeconds: diffInSeconds,  
      mathMax: Math.max(diffInSeconds * 1000 * 30, 10),  
    });  
    await this.cacheManager.set(  
      tokenKey,  
      payload,  
      Math.max(diffInSeconds * 1000 * 30, 10), //계산시간 고려 최소 1ms   
    );  
	return true;  
	}
 }

//미들웨어
export class BearerTokenMiddleware implements NestMiddleware {  
  constructor(  
    private readonly jwtService: JwtService,  
    private readonly configService: ConfigService,  
    @Inject(CACHE_MANAGER)  
    private readonly cacheManager: Cache,  
  ) {}
	...
	//유효성 검증한 토큰
	const tokenKey = `TOKEN_${token}`;  

	//블락한 토큰을 불러온다.
	const blockedToken = 
	await this.cacheManager.get(`BLOCKED_TOKEN_${token}`);  
	
	if (blockedToken) {  
	  throw new UnauthorizedException('차단된 토큰입니다.');  
	}

	//저장한 캐싱값이 있다면 바로 불러온다. 그는 
	const cachedPayload = await this.cacheManager.get(tokenKey);    
	if (cachedPayload) {  
	  req['user'] = cachedPayload;  
	  return next();  
	}
	
	const payload = '토큰 유효성 검증 및 디코딩 완료값'
	
	//저장할 캐시 키 지정
	const tokenKey = `TOKEN_${token}`;
	
	//payload 캐싱
	const expiryDate = +new Date(payload['exp'] * 1000);  
	const now = +Date.now();  
	const diffInSeconds = (expiryDate - now) / 1000;  
	await this.cacheManager.set(  
	  tokenKey,  
	  payload,
	  
	  //계산시간 고려 토큰 만료시간에서 30초 뺀다. 최소 1ms 지속한다.;
	  Math.max((diffInSeconds - 30) * 1000, 1),
	)
	
	req['user'] = payload
	next()
}
```