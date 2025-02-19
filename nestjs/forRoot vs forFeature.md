NestJS에서 forRoot와 forFeature는 모듈을 설정하고 주입하는 데 사용되는 패턴입니다. 각각의 역할과 차이를 이해하면 NestJS에서 효율적으로 모듈을 구성할 수 있습니다.

## **1. forRoot()**
• **애플리케이션의 루트 모듈에서 사용**
• **글로벌한 설정을 포함** (예: 데이터베이스 연결, 전역적으로 필요한 설정)
• 보통 **싱글톤 인스턴스를 생성**하여 애플리케이션 전체에서 공유됨
### **예제: TypeORM 설정**
```
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'test',
      autoLoadEntities: true,
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

- 위의 forRoot()는 TypeOrmModule을 전역적으로 설정하여 애플리케이션 전체에서 데이터베이스 연결을 공유하도록 합니다.

## **2. forFeature()**
• **특정 모듈에서 필요한 리포지토리 또는 기능을 설정**
• **도메인 단위의 기능을 주입** (예: 특정 엔티티의 레포지토리만 제공)
• forRoot()가 전역적인 설정을 다룬다면, forFeature()는 **도메인(기능)별 설정**을 다룸

### **예제: 특정 엔티티 리포지토리 설정**
```node
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  imports: [TypeOrmModule.forFeature([User])], // 특정 엔티티(User)만 등록
  providers: [UserService],
  controllers: [UserController],
})
export class UserModule {}
```
- forFeature([User])를 사용하면 UserRepository를 UserService에서 주입받아 사용할 수 있습니다.

## **3. forRoot() vs forFeature() 차이점**

| **비교 항목** | forRoot()                    | forFeature()                       |
| --------- | ---------------------------- | ---------------------------------- |
| 사용 위치     | AppModule 등 최상위 모듈           | 특정 도메인 모듈                          |
| 목적        | 전역적인 설정 (예: DB 연결)           | 도메인별 리포지토리 또는 기능 추가                |
| 싱글톤 여부    | 예 (앱 전체에서 공유)                | 아니요 (모듈별 인스턴스)                     |
| 예제        | TypeOrmModule.forRoot({...}) | TypeOrmModule.forFeature([Entity]) |

## **4. 정리**
• forRoot()는 **애플리케이션 전역 설정**을 위한 것 (ex. 데이터베이스, 설정 값)
• forFeature()는 **특정 모듈에서만 필요한 기능**을 제공하는 것 (ex. 특정 엔티티의 리포지토리)