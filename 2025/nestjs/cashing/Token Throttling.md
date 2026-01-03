- 특정 유저가 특정 경로에 접근횟수를 제한하기
- 메모리에서 특정 사용자의 특정 endpoint 접근 횟수 확인하고 제한하기
- 컨트롤러에 데코레이터를 이용해 해당 경로의 접근횟수 제한을 적용한다.
	- 특정 시간동안 특정 횟수의 요청 적용
- 인터셉터에서 데코레이터에 적용할 기능을 생성한다.
	- 데코레이터를 불러온다.
		- 없다면 Throttling 대상이 아니므로 건너뛴다.
	- 데코레이터가 있다면 캐싱 여부를 확인한다.
		- 캐싱이 있는 경우
			- 값이 요청횟수 미만이면 +1 count
			- 값이 요청횟수에 도달하면 에러 발생
		- 캐싱이 없는 경우, 새롭게 생성한다.
		  
- 저장할 키는 method_path_userId_minute로 지정한다.

```node
//데코레이터
export const Throttle = Reflector.createDecorator<{  
  count: number;  
  unit: 'minute';  
}>();

//인터셉터
import {  
  CallHandler,  
  ExecutionContext,  
  ForbiddenException,  
  Inject,  
  Injectable,  
  NestInterceptor,  
} from '@nestjs/common';  
import { Cache, CACHE_MANAGER } from '@nestjs/cache-manager';  
import { Reflector } from '@nestjs/core';  
import { Observable, tap } from 'rxjs';  
import { Throttle } from '../decorator/throttle.decorator';  
  
@Injectable()  
export class ThrottleInterceptor implements NestInterceptor {  
  constructor(  
    @Inject(CACHE_MANAGER)  
    private readonly cacheManager: Cache,  
    private readonly reflector: Reflector,  
  ) {}  
  
  async intercept(  
    context: ExecutionContext,  
    next: CallHandler<any>,  
  ): Promise<Observable<any>> {  
    const request = context.switchToHttp().getRequest();  
  
    // ket: URL_USERID_MINUTE  
    // value: count    
	const userId = request?.user?.sub;  
    if (!userId) {  
      return next.handle();  
    }  
	
	//데코레이터 값 불러오기
    const throttleOptions = this.reflector.get<{  
      count: number;  
      unit: 'minute';  
    }>(Throttle, context.getHandler());
	
	//데코레이터 지정이 안되있으니 건너뛴다.
    if (!throttleOptions) {  
      return next.handle();  
    }  
  
    const date = new Date();  
    const minute = date.getMinutes();  
    const key = `${request.method}_${request.path}_${userId}_${minute}`;  
	
    const count = await this.cacheManager.get<number>(key);  
    console.log('count', count);  
  
    if (count && count >= throttleOptions.count) {  
      throw new ForbiddenException('요청횟수 초과');  
    }  
  
    return next.handle().pipe(  
      tap(async () => {  
        const count = (await this.cacheManager.get<number>(key)) ?? 0;  
        await this.cacheManager.set(key, count + 1, 60000);  
      }),  
    );  
  }  
}
```

- 컨트롤러에서 @Throttle()을 이용해 특정 시간동안의 접근횟수를 지정할 수 있다.
```node
@Get()  
@Public()  
@Throttle({ count: 5, unit: 'minute' })  
getMovies(@Query() dto: GetMoviesDto, @UserId() userId: number) {  
  return this.movieService.findAll(dto, userId);  
}
```