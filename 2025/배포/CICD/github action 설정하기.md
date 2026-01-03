## AWS 사용자권한 생성
- elastic beanstalk 서버를 사용하는 것으로 가정한다.
- 사용자 권한정책은 elastic beanstalk와 s3 권한으로 생성한다
	- AdministratorAccess-AWSElasticBeanstalk
	- AmazonS3FullAccess
- 사용자 생성 완료되면 access key를 생성한다.
	![[Pasted image 20250227143046.png|400]]

### github 설정
- github > my repository > setting > Secret and variables > action > new action secret 선택
- 3가지 secret을 만든다. 이 권한으로 aws에 접근할 . 수있따.
	- AWS_ACCESS_KEY_ID: 생성한 사용자권한의 **access key**
	- AWS_REGION: **ap-northeast-2**
	- AWS_SECRET_ACCESS_KEY: 생성한 사용자권한의 **Secret access key**
		![[Pasted image 20250227143159.png|500]]

### 프로젝트 yml 파일 생성
- 프로젝트 루트 폴더에 .github/workflows/deploy.yml생성

```yml
name: Deploy to AWS Elastic Beanstalk  

# main 브랜치에 코드가 push될 때만 실행
on:  
  push:  
    branches:  
      - main  
# ubuntu-latest 환경에서 실행
jobs:  
  build-and-deploy:  
    runs-on: ubuntu-latest  
  
    steps:
    # GitHub 저장소의 코드를 가져온다.
      - name: Checkout Code  
        uses: actions/checkout@v3

    # Node.js **22 버전**을 설치
      - name: Set up NodeJS  
        uses: actions/setup-node@v3  
        with:  
          node-version: '22'  

	# npm i를 실행하여 필요한 패키지를 설치
      - name: Install Dependencies  
        run: npm i  

    # npm run build를 실행하여 프로젝트를 빌드
      - name: Build Project  
        run: npm run build  

	# 배포 파일 압축
      - name: Zip Artifact For Deployment  
        run: zip -r deploy.zip .  

	# AWS 인증 정보secrets에 저장된 키)로 AWS S3와 연결
	# 압축된 deploy.zip을 S3 버킷에 업로드
      - name: Upload To S3  
        env:  
          AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}  
          AWS_SECRET_ACCESS_KEY: ${{secrets.AWS_SECRET_ACCESS_KEY}}  
          AWS_REGION: ${{secrets.AWS_REGION}}  
        run: |  
          aws configure set region $AWS_REGION  
          aws s3 cp deploy.zip s3://nestjs-presigned-url-test-bucket/deploy.zip  

	# AWS Elastic Beanstalk 배포
      - name: Deploy To Aws Elastic Beanstalk  
        env:  
          AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}  
          AWS_SECRET_ACCESS_KEY: ${{secrets.AWS_SECRET_ACCESS_KEY}}  
          AWS_REGION: ${{secrets.AWS_REGION}}  
        run: |
		# deploy.zip을 기반으로 새로운 애플리케이션 버전 생성
		# version-label로 현재 GITHUB_SHA(커밋 SHA 값)를 사용
          aws elasticbeanstalk create-application-version \  
            --application-name "project name"  
            --version-label $GITHUB_SHA \  
            --source-bundle S3Bucket="nestjs-presigned-url-test-bucket",S3Key="deploy.zip"  

		# 생성한 애플리케이션 버전을 기존 Elastic Beanstalk 환경에 배포	
          aws elasticbeanstalk update-environment \  
            --application-name "생성한 서버 이름"  
            --environment-name "생성한 서버 환경이름"  
            --version-label $GITHUB_SHA
```