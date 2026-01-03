NestJS에서 기본 제공하는 Logger를 사용하면 콘솔에 로그를 출력할 수 있지만, 로그를 파일에 저장하려면 **winston 또는 pino 같은 로깅 라이브러리**를 활용하는 것이 일반적입니다. 여기서는 **winston과 nest-winston을 사용하여 로그 파일을 생성하는 방법**을 설명하겠습니다.

## **1. winston 및 nest-winston 설치**
- 먼저 필요한 패키지를 설치합니다.

```
pnpm install winston nest-winston
```

## **2. 로거 모듈(LoggerModule) 생성**
- NestJS에서 로그를 관리할 수 있도록 **전역 로거 모듈**을 설정합니다.

**logger.module.ts**
```node
import { Module } from '@nestjs/common';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

@Module({
  imports: [
    WinstonModule.forRoot({
      transports: [
        // 콘솔에 로그 출력
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.colorize(),
            winston.format.printf(({ level, message, timestamp }) => {
              return `[${timestamp}] ${level}: ${message}`;
            }),
          ),
        }),
        // 파일에 로그 저장 (application.log)
        new winston.transports.File({
          filename: 'logs/application.log',
          level: 'info', // info 이상 레벨 로그 저장
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json(),
          ),
        }),
        // 에러 로그 저장 (error.log)
        new winston.transports.File({
          filename: 'logs/error.log',
          level: 'error', // error 이상 레벨 로그 저장
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json(),
          ),
        }),
      ],
    }),
  ],
  exports: [WinstonModule], // 다른 모듈에서 사용할 수 있도록 export
})
export class LoggerModule {}
```

## **3. AppModule에서 LoggerModule 적용**
- 이제 애플리케이션의 루트 모듈에서 LoggerModule을 불러옵니다.
**app.module.ts**
```node
import { Module } from '@nestjs/common';
import { LoggerModule } from './logger.module';

@Module({
  imports: [LoggerModule],
})
export class AppModule {}
```

## **4. 서비스에서 로거 사용하기**
NestJS의 LoggerService를 주입받아 사용할 수 있습니다.
**app.service.ts**
```node
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class AppService {
  private readonly logger = new Logger(AppService.name);

  getHello(): string {
    this.logger.log('Hello World 요청됨'); // info 레벨 로그 (logs/application.log 저장됨)
    this.logger.warn('경고 메시지 테스트'); // warn 레벨 로그
    this.logger.error('에러 발생!'); // error 레벨 로그 (logs/error.log 저장됨)
    return 'Hello World!';
  }
}
```

## **5. 실행 및 로그 확인**

NestJS를 실행한 후 logs/ 폴더에서 로그 파일을 확인하면 됩니다.
```json
**로그 파일 예시 (logs/application.log)**
{"timestamp":"2025-02-19T12:00:00.000Z","level":"info","message":"Hello World 요청됨"}
{"timestamp":"2025-02-19T12:00:01.000Z","level":"warn","message":"경고 메시지 테스트"}

```

```json
**로그 파일 예시 (logs/error.log)**
{"timestamp":"2025-02-19T12:00:02.000Z","level":"error","message":"에러 발생!"}

```

## **추가: log 디렉토리 자동 생성**
NestJS 실행 시 logs/ 디렉토리가 없으면 자동 생성하도록 수정할 수도 있습니다.

```node
import * as fs from 'fs';
import * as path from 'path';

const logDir = path.join(__dirname, '..', 'logs');
if (!fs.existsSync(logDir)) {
  fs.mkdirSync(logDir);
}
```
- 이 코드를 logger.module.ts 상단에 추가하면, logs/ 폴더가 없을 경우 자동으로 생성됩니다.

