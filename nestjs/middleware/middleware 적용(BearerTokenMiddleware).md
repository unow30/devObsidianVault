- 'bearer $token'을 받으면 payload를 return하는 middleware 생성
- 'basic $token'을 받는 route는 middleware를 exclude한다
- [[5. token 발급, 재발급하기]]등 일부 코드 수정된다.
#### BearerTokenMiddleware 생성
- headers.authorization을 가져온다.
	- authorization이 없으면 next()한다.
- 'bearer $token'을 분리하여 token만 가져온다.
- jwtService.decode()로 payload를 가져온다.
	- payload검증은 안한다. type이 refresh인지 access인지 확인한다.
	- 둘 다 아닌 경우엔 에러
- jwtService.verifyAsync()로 payload추출 및 검증한다.
	- secret을 이용해 token의 payload가 유효한지 검증까지 한다.
	- 유효한 payload라면 해당값을 req.user에 담고 next()한다.
```node
import {  
  BadRequestException,  
  Injectable,  
  NestMiddleware,  
  UnauthorizedException,  
} from '@nestjs/common';  
import { NextFunction } from 'express';  
import { JwtService } from '@nestjs/jwt';  
import { ConfigService } from '@nestjs/config';  
import { envVariableKeys } from '../../common/const/env.const';  
  
@Injectable()  
export class BearerTokenMiddleware implements NestMiddleware {  
  constructor(  
    private readonly jwtService: JwtService,  
    private readonly configService: ConfigService,  
  ) {}  
  async use(req: Request, res: Response, next: NextFunction) {  
    /**  
     * basic $token 를 받는 route는 exclude 한다.  
     * bearer $token 을 받는 route 는 middleware 를 거친다.  
     */    
     const authHeader = req.headers['authorization'];  
  
    if (!authHeader) {  
      next();  
      return;  
    }  
  
    const token = this.validateBearerToken(authHeader);  
  
    try {  
      //디코드만 하고 검증은 안한다.  
      const decodedPayload = this.jwtService.decode(token);  
      if (  
        decodedPayload.type !== 'refresh' &&  
        decodedPayload.type !== 'access'  
      ) {  
        throw new UnauthorizedException('잘못된 토큰입니다.');  
      }  
  
      //payload 가져오며 검증도 한다.  
      const payload = await this.jwtService.verifyAsync(token, {  
        secret: this.configService.get<'string'>(  
          decodedPayload.type === 'refresh'  
            ? envVariableKeys.refreshTokenSecret  
            : envVariableKeys.accessTokenSecret,  
        ),  
      });  

	  //payload값이 req.user에 담긴다.
      req['user'] = payload;  
      next();  
    } catch (error) {  
      //error 에 따른 throw 설정 가능  
      console.log(error);  
      throw new UnauthorizedException('토큰이 만로되었습니다.');  
    }  
  }  
  
  validateBearerToken(rawToken: string) {  
    // console.log(rawToken);  
    const basicSplit = rawToken.split(' ');  
    if (basicSplit.length !== 2) {  
      throw new BadRequestException('토큰 포맷이 잘못되었습니다.');  
    }  
  
    const [bearer, token] = basicSplit;  
  
    if (bearer.toLowerCase() !== 'bearer') {  
      throw new BadRequestException('토큰 포맷이 잘못되었습니다.');  
    }  
  
    return token;  
  }  
}
```

#### AppModule에 middleware 적용
- NestModule 구현
	- consumer.apply(BearerTokenMiddleware) 적용
- exclude 설정
	- 'auth/login' POST, 'auth/register' POST는 'basic $token' 을 이용하므로 제외
- 모든 route에 적용
```node
// import 생략  
@Module({  
  //...생략
})  
export class AppModule implements NestModule {  
  configure(consumer: MiddlewareConsumer) {  
    consumer  
      .apply(BearerTokenMiddleware)  
      .exclude(  
        {  
          path: 'auth/login',  
          method: RequestMethod.POST,  
        },  
        {  
          path: 'auth/register',  
          method: RequestMethod.POST,  
        },  
      )  
      .forRoutes('*');  
  }  
}
```

#### auth Controller 수정

- 'auth/token/access'는 'bearer $token'(refresh token)을 받아 새로운 access token을 발급한다.
- BearerTokenMiddleware의 payload를 받기 위해 @Request()를 이용한다.

```node

//기존
@Post('token/access')  
async rotateAccessToken(@Headers('authorization') token: string) {  
  const payload = await this.authService.parseBearerToken(token, true);  
  return {  
    accessToken: await this.authService.issueToken(payload, false),  
  };  
}
   
//미들웨어 적용으로 인한 수정(BearerTokenMiddleware)
// req.user는 middleware에서 받아온 payload가 있다.
@Post('token/access')  
async rotateAccessToken(@Request() req) {  
  return {  
    accessToken: await this.authService.issueToken(req.user, false),  
  };  
}
```

#### 다른 컨트롤러에서도 payload를 받아오는지 확인
- 'movie' GET 요청시 headers.authorization에 tokens을 입력한다.
- 해당 컨트롤러에 @Request()를 추가하고 req.user를 콘솔로 확인한다.
```node
@Get()  
getMovies(  
  @Request() req: any,  
  @Query('title', MovieTitleValidationPipe) title?: string,  
) {  
  console.log(req.user);  
  return this.movieService.findAll(title);  
}
```