https://aws.amazon.com/ko/solutions/implementations/live-streaming-on-aws/\

![[Screenshot_20240923_174030_YouTube.jpg]]
AWS Elemental Link 디바이스에서 Real-Time Transport Protocol(RTP), Real-Time Messaging Protocol(RTMP), HTTP Live Streaming(HLS) 콘텐츠 또는 라이브 비디오를 수집하도록 이 솔루션을 구성할 수 있습니다.

**1단계  
**[AWS Elemental MediaLive](https://aws.amazon.com/ko/medialive/)는 입력 피드를 수집하고 해당 콘텐츠를 1개의 적응형 비트레이트(ABR) HTTP 라이브 스트리밍(HLS) 스트림으로 출력으로 트랜스코딩합니다.

**2단계**  
[Amazon Simple Storage Service](https://aws.amazon.com/ko/s3/)(S3)는 인코딩된 세그먼트를 호스트할 수 있는 확장 가능한 고가용성 스토리지 버킷을 제공합니다.
**3단계**  
[Amazon CloudFront](https://aws.amazon.com/ko/cloudfront/) 배포는 **Amazon S3** 사용자 지정 엔드포인트를 원본으로 사용하도록 구성되어 있습니다. **CloudFront** 배포는 짧은 지연 시간과 빠른 전송 속도로 시청자에게 라이브 스트림을 전달합니다.  
 
**4단계**  
**S3** 버킷은 **CloudFront** 로그를 저장합니다.

