
## S3 사용자 권한 설정
- iam > 사용자 > 사용자 생성 > 이름 입력 > 권한 설정 > 직접 정책 연결 > AmazonS3FullAccess 선택
- 선택한 사용자 화면에서 엑세스 키 만들기 선택 > Application running outside AWS 
	![[Pasted image 20250225130026.png|500]]
	![[Pasted image 20250225130118.png|500]]
- 엑세스 키, 시크릿 엑세스 키 .csv파일 저장후 완료

## s3 버킷 생성
- 테스트 용으로 사용하고 제거할 것이기 때문에 다음과 같이 설정
	- 객체 소유권 > acl 활성화됨 access control list를 다른 계정에서도 사용할 수 있다.
- 이 버킷의 퍼블릭 액세스 차단 설정 모두 해제
	> 모든 퍼블릭 액세스 차단을 비활성화하면 이 버킷과 그 안에 포함된 객체가 퍼블릭 상태가 될 수 있습니다.
    > 정적 웹 사이트 호스팅과 같은 구체적으로 확인된 사용 사례에서 퍼블릭 액세스가 필요한 경우가 아니면 모든 퍼블릭 액세스 차단을 활성화하는 것이 좋습니다.    

- 버킷 버전 관리 비활성화
	- 과금 제한 목적
- 생성한 버킷 확인
## aws-sdk 설치
- `pnpm install aws-sdk`
## presignedUrl 코드 작성
- env 환경변수 추가
```
	# AWS  
	AWS_SECRET_ACCESS_KEY=XXXXX  
	AWS_ACCESS_KEY_ID=XXXXX  
	AWS_REGION=XXXXX  
	BUCKET_NAME=XXXXX
```
- ConfigService, joi에서 사용 가능하도록 설정
```node
//const.ts
//env의 키값만 들고 있는 파일
//...
const awsSecretAccessKey = 'AWS_SECRET_ACCESS_KEY';  
const awsAccessKeyId = 'AWS_ACCESS_KEY_ID';  
const awsRegion = 'AWS_REGION';  
const bucketName = 'BUCKET_NAME';

export const envVariableKeys = {  
 //......
	awsSecretAccessKey,  
	awsAccessKeyId,  
	awsRegion,  
	bucketName, 
};


//app.modue.ts
//...
// ConfigService에서 사용할 env정보 검증
ConfigModule.forRoot({  
  isGlobal: true, //모든 모듈에서 ConfigModule 사용 true  
  validationSchema: 
  Joi.object({  
   //...
    AWS_SECRET_ACCESS_KEY: Joi.string().required(),  
    AWS_ACCESS_KEY_ID: Joi.string().required(),  
    AWS_REGION: Joi.string().required(),  
    BUCKET_NAME: Joi.string().required(),  
  }),  
}),

//사용 예시
configService.get<string>(envVariableKeys.awsAccessKeyId)
```
- service 구현
```node
import * as AWS from 'aws-sdk';  
import { v4 as Uuid } from 'uuid';  
import { ConfigService } from '@nestjs/config';  
import { envVariableKeys } from './const/env.const';  
  
@Injectable()  
export class CommonService {  
  private s3: AWS.S3;  
  constructor(private readonly configService: ConfigService) {  
    AWS.config.update({  
      credentials: {  
        accessKeyId: configService.get<string>(
          envVariableKeys.awsAccessKeyId,
        ),  
        secretAccessKey: configService.get<string>(  
          envVariableKeys.awsSecretAccessKey,  
        ),  
      },  
      region: configService.get<string>(
        envVariableKeys.awsRegion
      ),  
    });  
  
    this.s3 = new AWS.S3();  
  }

  async createPresignedUrl(expiresIn = 300) {  
	  const params = {  
	    Bucket: this.configService.get<string>(envVariableKeys.bucketName),  
	    //버킷에 생성될 파일명, 경로  
	    Key: `temp/${Uuid()}.mp4`,  
	    ExpiresIn: expiresIn,  
	    // 보두가 읽을 수 있음  
	    ACL: 'public-read',  
	  };  
	  
	  try {  
	    const url = await this.s3.getSignedUrlPromise('putObject', params);  
	    return url;  
	  } catch (e) {  
	    console.log(e);  
	    throw new InternalServerErrorException('S3 Presigned Url error');  
	  }  
  }
}


@Controller('common')  
export class CommonController {  
  constructor(private readonly commonService: CommonService) {}  
  
  @Post('/upload/s3')  
  async createPresignedUrl() {  
    return {  
      url: await this.commonService.createPresignedUrl(),  
    };  
  }
}
```

## s3 업로드 테스트
- 코드 실행 후 postman으로 요청하기
	- 파일을 전송하지 않았다. 요청하면 파일을 전송할 수 있는 url을 전달해준다.
		![[Pasted image 20250225135944.png|500]]
	- 이 url과 put 요청을 이용해서 파일을 업로드한다.
		- binary 형식으로 파일을 업로드한다.
		- 성공시 200 ok, 1 true를 응답받는다.
		- 버킷에 temp폴더에 파일이 들어있다.
		![[Pasted image 20250225140632.png]]
		![[Pasted image 20250225140748.png|500]]
