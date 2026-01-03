- 다음과 같은 로그(warning) 발생
- 로그를 쌓으려면 어딘가에 저장해야한다. (파일, db, 이메일, http 요청 등)
- 이런 로그를 저장하는 매개체를 transports라고 부르며 이를 지정하지 않아 생긴 에러
> [winston] Attempt to write logs with no **transports**, which can increase memory usage: {"context":"RoutesResolver","level":"info","message":"SomeController {/some_route}:"}

- winston 모듈을 사용하기 위해선 기존 logger에 inject해야한다.
- winston의 기본 메시지 출력은 json이다.

```node
import { Inject, Injectable, Logger, LoggerService } from '@nestjs/common';  
import { WINSTON_MODULE_NEST_PROVIDER } from 'nest-winston';  

@Injectable()  
export class TasksService {  
  constructor(  
    ...,
	// 기본 logger 기능에 winston모듈을 주입  
    @Inject(WINSTON_MODULE_NEST_PROVIDER)  
    private readonly logger: LoggerService,  
  ) {}
	  foo(){
		  const err = new Error('에러 발생')
		//winston은 fatal level을 제공하지 않는다.
		//this.logger.fatal('fatal level', null, TasksService.name);  
		this.logger.error('error level', err.stack, TasksService.name);  
		this.logger.warn('warn level', TasksService.name);  
		this.logger.log('log level', TasksService.name);  
		this.logger.debug('debug level', TasksService.name);  
		this.logger.verbose('verbose level', TasksService.name);
	}
  }
```


```node
export interface winston.ConsoleTransportOptions extends TransportStreamOptions {
	consoleWarnLevels?: string[]
	stderrLevels?: string[]
	debugStdout?: boolean
	eol?: string
	forceConsole?: boolean
}
```


winston module config
```node
WinstonModule.forRoot({  
  level: 'debug',  
  transports: [  
	//로그 콘솔 표시하기
    new winston.transports.Console({  
      format: winston.format.combine(  
        winston.format.timestamp(),  
        winston.format.colorize({ all: true }), 
        //커스텀 로그 생성 
        winston.format.printf(  
          ({ timestamp, context, level, message, stack }) => {  
            const logMessage = `${timestamp} ${context} ${level} ${message}`;
            // 스택이 있으면 추가
            return stack ? `${logMessage}\n${stack}` : logMessage; 
            
          },  
        ),  
      ),  
    }),
      
    //로그 파일에 저장하기
    new winston.transports.File({  
      dirname: join(process.cwd(), 'logs'),  
      filename: 'logs.log',  
      format: winston.format.combine(  
        winston.format.timestamp(),  
        //커스텀 로그 생성
        winston.format.printf(  
          ({ timestamp, context, level, message, stack }) => {  
            const logMessage = `${timestamp} ${context} ${level} ${message}`;  
            // 스택이 있으면 추가  
            return stack ? `${logMessage}\n${stack}` : logMessage; 
          },  
        ),  
      ),  
    }),  
  ],  
}),
```