https://aws.amazon.com/ko/solutions/implementations/live-streaming-on-aws/\

![[Screenshot_20240923_174030_YouTube.jpg]]
AWS Elemental Link 디바이스에서 Real-Time Transport Protocol(RTP), Real-Time Messaging Protocol(RTMP), HTTP Live Streaming(HLS) 콘텐츠 또는 라이브 비디오를 수집하도록 이 솔루션을 구성할 수 있습니다.


live streaming on aws with amazon s3

![Live Streaming on AWS with Amazon S3 아키텍처 흐름 다이어그램](https://d1.awsstatic.com/Solutions/Solutions%20Category%20Template%20Draft/Solution%20Architecture%20Diagrams/live-streaming-on-aws-with-amazon-s3.4b28f532989f01822d431e632858d4f39e4a2cec.png "Live Streaming on AWS with Amazon S3 | 아키텍처 흐름 다이어그램")**1단계  
**[AWS Elemental MediaLive](https://aws.amazon.com/ko/medialive/)는 입력 피드를 수집하고 해당 콘텐츠를 1개의 적응형 비트레이트(ABR) HTTP 라이브 스트리밍(HLS) 스트림으로 출력으로 트랜스코딩합니다.

**2단계**  
[Amazon Simple Storage Service](https://aws.amazon.com/ko/s3/)(S3)는 인코딩된 세그먼트를 호스트할 수 있는 확장 가능한 고가용성 스토리지 버킷을 제공합니다.
**3단계**  
[Amazon CloudFront](https://aws.amazon.com/ko/cloudfront/) 배포는 **Amazon S3** 사용자 지정 엔드포인트를 원본으로 사용하도록 구성되어 있습니다. **CloudFront** 배포는 짧은 지연 시간과 빠른 전송 속도로 시청자에게 라이브 스트림을 전달합니다.  
 
**4단계**  
**S3** 버킷은 **CloudFront** 로그를 저장합니다.


**적응형 비트 전송률(ABR)**

HTTP 네트워크를 통한 비디오 스트리밍을 개선하기 위해 네트워크 상황에 따라 비디오 품질을 조정하는 스트리밍 방법입니다.

**HTTP 라이브 스트리밍(HLS)**

Apple Inc.에서 개발한, 인터넷을 통해 미디어를 전송하는 HTTP 기반 스트리밍 프로토콜입니다.

**HTTP를 통한 동적 적응형 스트리밍(DASH)**

HTTP 기반 스트리밍 프로토콜(MPEG-DASH라고도 함)은 인터넷을 통해 미디어를 전송하며 MPEG(Motion Picture Experts Group)에서 개발되었습니다.

**공통 미디어 애플리케이션 형식(CMAF)**

인터넷을 통한 미디어 전송을 개선하기 위한 HTTP 기반 스트리밍 및 패키징 표준입니다. HLS 및 DASH와 호환되며 Apple과 Microsoft에서 공동 개발했습니다.