## pnpm 설치
`pnpm i @nestjs/cache-manager cache-manager`

## 모듈에 CacheModule 적용
```node
import { CacheModule } from '@nestjs/cache-manager';

@Module({  
  imports: [  
    ..., 
    CacheModule.register({}),  
  ],  
  controllers: [MovieController],  
  providers: [MovieService],  
})  
export class MovieModule {}
```
-  `'@nestjs/common/cache'`;의 cache module이 아니다. (duplicated)

## 캐싱을 불러오는 컨트롤러 경로 생성

### 1) 캐싱 없이 직접 불러오기
```node

//controller
//movie/recent  
@Get('recent')  
getMovieRecent() {  
  return this.movieService.findRecent();  
}

//service
//최신영화 10개만 불러온다.  
async findRecent() {  
  return await this.movieRepository.find({  
    order: {  
      createdAt: 'DESC',  
    },  
    take: 10,  
  });  
}
```

### 2) 캐싱 테스트
- 응답값을 캐시에 저장한다.
- 저장된 캐시값이 있다면 바로 불러온다.
- 서버 재시작 전까지 캐시는 저장된다.
```node
import { CACHE_MANAGER, Cache } from '@nestjs/cache-manager';  
  
@Injectable()  
export class MovieService {  
  constructor(  
	...,
    @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,  
  ) {}  
  
  //최신영화 10개만 불러온다.  
  async findRecent() {  
    const cacheData = await this.cacheManager.get('MOVIE_RECENT');
    if(cacheData){
    console.log('캐시 가져옴')
	    return cacheData
    }
  
    const data = await this.movieRepository.find({  
      order: {  
        createdAt: 'DESC',  
      },  
      take: 10,  
    });  

	await this.cacheManager.set('MOVIE_RECENT', data)
	return data
  }
```

- 서버 실행후 첫 요청: GET /movie/recent 31ms 캐시요청 아님
- 다음 요청: GET /movie/recent 0ms 캐시 가져옴. ms도 많이 줄어듬

## TTL(Time to Live)
- 캐시가 얼마나 저장할지 시간을 지정한다.
- TTL 0이라면 내용을 지우지 않고 무한히 저장한다.
- 기본값은
```node
await this.cacheManager.set('MOVIE_RECENT', data, 0)
```

- 서버 재시작 후 요청: GET /movie/recent 16ms
- 다음 요청: POST /auth/token/access 1ms

- TTL 1(ms)로 지정한 다음 실행
```node
await this.cacheManager.set('MOVIE_RECENT', data, 1) //ms
```
- 서버 재시작 후 요청: GET /movie/recent 17ms
- 다음 요청: GET /movie/recent 25ms 캐시 불러오기가 아니다.

- module에서 직접 요청할 수 있다.
	`CacheModule.register({ ttl: 3000 }),`
- module ttl보다 service에서의 ttl 설정이 우선된다.(override)

## cache Interceptor
- cache-manager에서 CacheInterceptor를 불러온다.
- interceptor가 해당 요청의 결과를 자동으로 캐싱한다. (url 기반으로 캐싱된다.)
- 해당 경로를 연속으로 3번 불러본다.(TTL: 3000ms 설정)

```node
import { CacheInterceptor as CI } from '@nestjs/cache-manager';

//movie/recent  
@Get('recent')  
@UseInterceptors(CI)  
getMovieRecent() {  
console.log('getMoviesRecent() 실행')
  return this.movieService.findRecent();  
}
```
- 첫번째 요청
	getMoviesRecent() 실행 GET /movie/recent 3ms
- 두번째 요청
	GET /movie/recent 1ms
- 3번째 요청
	GET /movie/recent 1ms

- url 기반으로 캐싱되므로 query parameter나 path parameter가 붙는 경우 다른 경로로 캐싱이 된다.
- 인터셉터가 지정된 위치에 따라 캐싱이 이루어진다. 컨트롤러 전체를 캐싱할 수도 있다.

## @CacheKey()
- 캐시 키값을 변경할 수 있다.
- @nestjs/cache-manager에서 가져온다.
- 캐싱이 해당 키에 일괄적으로 저장되므로 url에 지정되지 않는다.

```node
//movie/recent  
@Get('recent')  
@UseInterceptors(CI)  
@CacheKey('getMoviesRecent')  
getMovieRecent() {  
  console.log('getMoviesRecent() 실행');  
  return this.movieService.findRecent();  
}
```

## @CacheTTL(1000)
- 캐싱의 TTL을 지정한다. 이전에 지정된 TTL을 덮어씌운다.(override)

```node
//movie/recent  
@Get('recent')  
@UseInterceptors(CI)  
@CacheKey('getMoviesRecent')  
@CacheTTL(1000) //1초 캐싱
getMovieRecent() {  
  console.log('getMoviesRecent() 실행');  
  return this.movieService.findRecent();  
}
```