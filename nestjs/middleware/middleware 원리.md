### 설명
- middleware는 라우트 핸들러가 실행되기 전에 실행된다.
	- client => http request => middleware => route Handler @requestMapping()
	- route Handler @requestMapping()  => middleware => http response => client
- request와 response 객체에 접근할 수 있다.

### 특징
- 자유로운 코드 실행
- 요청, 응답객체 변경
- 요청, 응답 사이클 중단
- 다음 미들웨어 실행가능
- 비파괴적
	- 미들웨어로 인한 라우트 핸들러의 로직이 변경될 일은 없다.

### 선언
```node

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
	use(req, res, next){
		//미들웨어에서 구현할 로직 작성
		console.log('logging something')
		next()// 완료시 다음 미들웨어나 라우트 핸들러로 이동
	}
}
```


### 적용법
#### module에 선언
- NestModule을 implement한다.
	- NestModule의 configure 메소드를 구현한다.
	- customer로 적용할 미들웨어와 라우트 범위를 지정할 수 있다.
- 어떤 route에 적용할지 명칭을 입력한다.
	- 모든 곳에 적용하려면 와일드카드 * 를 입력한다.
```node
import {Module, NestModule, MiddlewareConsumer} from '@nestjs/common'
import {LoggerMiddleware} from './common/middleware/logger.middleware';
import {SomeModule} from './some/some.module'

@Module({
	imports: [SomeModule]
})
export class AppModule implement NestModule {
	configure(customer: MiddlewareCustomer){
		consumer
			.apply(LoggerMiddleware)
			.forRoutes('some')
	}
}
```

- forRoutes에서 requestMethod또한 지정할 수 있다.
	- some 라우트의 GET요청에만 LoggerMiddleware가 적용된다.
```node
import {Module, NestModule,RequestMethod, MiddlewareConsumer} from '@nestjs/common'
import {LoggerMiddleware} from './common/middleware/logger.middleware';
import {SomeModule} from './some/some.module'

@Module({
	imports: [SomeModule]
})
export class AppModule implement NestModule {
	configure(customer: MiddlewareCustomer){
		consumer
			.apply(LoggerMiddleware)
			.forRoutes({path: 'some', method:RequestMethod.GET})
	}
}
```

- exclude를 이용해 특정 route만 제외할 수 있다.
- 여러 경로를 지정할 수 있다.
- 와일드카드를 이용할 수 있다.
```node
import {Module, NestModule,RequestMethod, MiddlewareConsumer} from '@nestjs/common'
import {LoggerMiddleware} from './common/middleware/logger.middleware';
import {SomeModule} from './some/some.module'
import {ThisModule} from './this/this.module'
import Somecontroller

@Module({
	imports: [SomeModule, ThisModule, ThatModule]
})
export class AppModule implement NestModule {
	configure(customer: MiddlewareCustomer){
		consumer
			.apply(LoggerMiddleware)
			.exclude(
				{path: 'this', method: RequestMethod.GET},
			)
			//some route의 모든 경로에 middleware 적용
			//this route에서 get은 middleware제외
			.forRoutes(SomeController, ThisController)
	}
}
```

- 여러 미들웨어를 적용하기
```node
import {Module, NestModule,RequestMethod, MiddlewareConsumer} from '@nestjs/common'
import {LoggerMiddleware} from './common/middleware/logger.middleware';
import {customMiddleware} from './common/middleware/custom.middleware'
import {SomeModule} from './some/some.module'

@Module({
	imports: [SomeModule]
})
export class AppModule implement NestModule {
	configure(customer: MiddlewareCustomer){
		consumer
			//적은 순서대로 미들웨어가 적용한다.
			.apply(LoggerMiddleware, customMiddleware)
			.forRoutes({path: 'some', method:RequestMethod.GET})
	}
}

``` 

