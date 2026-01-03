## 1. **화질별 파일 생성 필요성**
- **필수 요구사항**: HLS의 적응형 비트레이트(ABR) 기능을 구현하려면 **반드시 화질별로 m3u8/ts 파일을 별도 생성**해야 합니다.    
    - 각 화질(예: 1080p, 720p, 480p)마다 인코딩된 비디오 세그먼트(ts)와 해당 화질을 지정하는 미디어 플레이리스트(m3u8)가 필요합니다[1](https://www.fastpix.io/blog/a-complete-guide-to-m3u8-files-in-hls-streaming)[4](https://en.wikipedia.org/wiki/HTTP_Live_Streaming)6.
        
    - 마스터 플레이리스트(master.m3u8)에서 각 화질의 대역폭, 해상도, 코덱 정보를 명시하여 플레이어가 네트워크 상태에 따라 자동 전환할 수 있게 합니다[5](https://stackoverflow.com/questions/71913543/how-to-generate-multiple-resolutions-hls-using-ffmpeg-for-live-streaming)6.

## 2. **저장 공간 증가 문제**
- **데이터 용량 증가**: 100MB 원본 MP4를 3개 화질(1080p/720p/480p)로 변환할 경우, **원본 대비 3~5배 이상의 저장 공간**이 필요할 수 있습니다[3](https://docs.dyntube.com/faqs/hls-streaming-and-storage)6.
    - **이유**:
        - 각 화질별 비트레이트가 다르며, 고화질일수록 데이터량 증가(예: 1080p=8Mbps, 720p=4Mbps, 480p=2Mbps).
        - 오디오 트랙, 자막, DRM 메타데이터 등 추가 데이터 포함.
            
    - **예시**:       
        - 원본 100MB → 1080p(120MB) + 720p(60MB) + 480p(30MB) = **총 210MB** (원본 대비 2.1배).
            

## 3. **최적화 방안**

- **CDN 캐싱 활용**: 자주 요청되는 세그먼트(ts)를 CDN 엣지 서버에 캐싱하여 저장 부하 분산[2](https://www.dacast.com/blog/http-live-streaming/)[4](https://en.wikipedia.org/wiki/HTTP_Live_Streaming).
    
- **지능형 인코딩 전략**:
    - **Tiered Encoding**: 주요 화질(1080p/720p)만 생성하고, 저화질(480p 이하)은 실시간 트랜스코딩(on-the-fly)으로 처리6.
        
    - **Per-Title Encoding**: 콘텐츠 특성에 맞춰 화질/비트레이트를 동적으로 조정하여 불필요한 고화질 파일 생성을 줄임[3](https://docs.dyntube.com/faqs/hls-streaming-and-storage).
        
- **저장 포맷 최적화**:
    - Fragmented MP4(fMP4) 사용으로 ts 대비 5~10% 저장 공간 절약 가능[4](https://en.wikipedia.org/wiki/HTTP_Live_Streaming).
        
    - HEVC/H.265 코덱 적용 시 동일 화질에서 H.264 대비 50% 용량 감소(단, 호환성 확인 필요).
        

## 4. **장단점 비교**

|구분|장점|단점|
|---|---|---|
|**화질별 저장**|ABR 지원으로 최적 재생 품질 보장|저장 공간 다중 소모|
|**단일 화질**|저장 공간 효율적|네트워크 불안정 시 버퍼링 발생|

## 5. **결론**

- **대규모 서비스**: 화질별 파일을 미리 생성해 CDN에 배포하는 것이 표준 접근법입니다. 저장 공간 증가는 ABR로 인한 대역폭 절감 효과로 상쇄 가능[2](https://www.dacast.com/blog/http-live-streaming/)[3](https://docs.dyntube.com/faqs/hls-streaming-and-storage).
    
- **소규모/특수 케이스**: 실시간 트랜스코딩(예: AWS MediaConvert, FFmpeg)을 활용해 저장 부하를 줄일 수 있지만, 인코딩 지연 시간이 발생할 수 있습니다6.
    
- **밸런스 권장**: 대상 사용자의 네트워크 환경(모바일/데스크탑 비율)과 콘텐츠 특성(라이브/VOD)에 따라 화질 수를 최적화하는 것이 중요합니다.


---
## 실시간 트랜스코딩(On-the-fly Transcoding)과 Per-Title Encoding 설명

## **실시간 트랜스코딩(On-the-fly Transcoding)**

- **정의**: 실시간 트랜스코딩은 영상 파일을 미리 변환해 두지 않고, 사용자의 요청이 들어올 때마다 서버가 영상을 작은 조각(세그먼트) 단위로 실시간 변환하여 스트리밍하는 방식입니다[1](https://www.coconut.co/articles/onthefly-video-transcoding-instant-magic)[4](https://patents.google.com/patent/US20150007237A1/en).
    
- **작동 방식**:
    
    - 영상이 업로드되면 서버는 전체 영상을 미리 변환하지 않고, 시청자가 요청하는 구간(혹은 세그먼트)만을 즉시 변환합니다.
        
    - 변환된 첫 번째 세그먼트가 준비되면 바로 스트리밍이 시작되고, 이후 세그먼트도 실시간으로 변환·전송됩니다[1](https://www.coconut.co/articles/onthefly-video-transcoding-instant-magic).
        
    - 네트워크 상태나 디바이스에 따라 필요한 포맷·화질로 변환할 수 있어, 다양한 환경에 유연하게 대응합니다.
        
- **장점**:
    
    - 시청자는 전체 변환이 끝날 때까지 기다리지 않고 곧바로 시청 가능.
        
    - 저장 공간 절약(모든 화질·포맷을 미리 저장하지 않아도 됨).
        
    - 네트워크 상황 변화에 실시간 대응 가능.
        
- **단점**:
    
    - 서버에 실시간 인코딩 부하가 큼.
        
    - 동시 접속자가 많거나 복잡한 영상일 경우 인코딩 지연이 발생할 수 있음.
        
    - 자주 요청되는 세그먼트는 캐싱해서 재사용할 수 있음[4](https://patents.google.com/patent/US20150007237A1/en).
        

## **Per-Title Encoding**

- **정의**: Per-Title Encoding은 영상마다 개별적으로 복잡도와 특성을 분석해, 해당 영상에 최적화된 인코딩 설정(비트레이트, 해상도 등)을 적용하는 방식입니다[2](https://bitmovin.com/encoding-service/per-title-encoding/)[3](https://www.fastpix.io/blog/per-title-encoding-for-streaming)[5](http://techblog.netflix.com/2015/12/per-title-encode-optimization.html)[6](https://www.ioriver.io/terms/per-title-encoding)[7](https://www.linkedin.com/pulse/content-based-encoding-per-title-sudeep-kumar)[8](https://www.gumlet.com/learn/per-title-encoding/)[9](https://cloudinary.com/glossary/per-title-encoding).
    
- **작동 방식**:
    
    - 영상의 움직임, 색상, 디테일, 장면 전환 등 복잡도를 자동 분석.
        
    - 분석 결과에 따라 각 영상에 맞는 최적의 비트레이트 래더(ABR ladder, 화질별 비트레이트 조합)를 생성합니다.
        
    - 예를 들어, 액션 영화(복잡도 높음)는 높은 비트레이트를, 애니메이션(복잡도 낮음)은 낮은 비트레이트를 할당해 효율적으로 인코딩합니다[5](http://techblog.netflix.com/2015/12/per-title-encode-optimization.html)[6](https://www.ioriver.io/terms/per-title-encoding)[7](https://www.linkedin.com/pulse/content-based-encoding-per-title-sudeep-kumar).
        
- **장점**:
    
    - 불필요한 데이터 낭비 없이 화질과 용량을 최적화.
        
    - 동일한 네트워크 환경에서 더 좋은 화질 제공, 혹은 같은 화질에서 더 적은 대역폭 사용.
        
    - 저장 공간 및 CDN 비용 절감, 사용자 경험 개선[2](https://bitmovin.com/encoding-service/per-title-encoding/)[3](https://www.fastpix.io/blog/per-title-encoding-for-streaming)[6](https://www.ioriver.io/terms/per-title-encoding)[8](https://www.gumlet.com/learn/per-title-encoding/).
        
- **단점**:
    
    - 영상마다 별도 분석·인코딩이 필요해 초기 인코딩 비용이 증가.
        
    - 구현 복잡도 증가.
        

## **비교 요약**

|구분|실시간 트랜스코딩(On-the-fly)|Per-Title Encoding|
|---|---|---|
|인코딩 시점|시청 요청 시 실시간 변환|영상 업로드 후 사전 분석 및 변환|
|저장 공간|최소화(필요한 부분만 변환/저장)|최적화된 여러 화질/비트레이트 저장|
|서버 부하|실시간 인코딩 부하 큼|초기 인코딩 부하, 이후 서비스는 가벼움|
|화질 최적화|네트워크/디바이스에 따라 실시간 대응|영상 특성에 맞춘 최적화된 비트레이트 래더|
|적용 예시|라이브 방송, VOD의 온디맨드 트랜스코딩|대형 OTT(Netflix 등) 라이브러리 VOD|

**요약**:

- **실시간 트랜스코딩**은 저장 공간은 절약하지만 서버 부하가 크고, 주로 라이브 방송이나 특수한 VOD에 사용됩니다.
    
- **Per-Title Encoding**은 영상별로 최적화된 화질·용량 조합을 제공해 대규모 스트리밍 서비스에서 효율성과 품질을 동시에 추구할 수 있습니다.


---

## AWS Elemental MediaConvert에서 **두 방법(온디맨드 실시간 트랜스코딩과 Per-Title 인코딩)** 모두 적용 가능합니다.

## 1. **온디맨드 실시간 트랜스코딩(On-the-fly Transcoding)**

- **지원 가능 여부**:
    
    - MediaConvert는 주로 파일 기반 콘텐츠를 대상으로 하는 서비스로, 사전 인코딩된 파일을 다양한 포맷과 해상도로 변환하는 데 최적화되어 있습니다.
        
    - **실시간 또는 온디맨드 요청 시, 인코딩 작업을 즉시 수행하는 기능**이 내장되어 있으며, 이를 통해 요청마다 실시간으로 트랜스코딩하는 것도 가능합니다.
        
    - 하지만, **이 방식은 서버에서 요청 시마다 인코딩을 수행하는 것으로, 일반적으로는 미리 인코딩된 세트(즉, 캐시된 파일)를 제공하는 것보다 비용과 시간이 더 소요**됩니다.
        
    - MediaConvert는 **즉시 요청에 대응하는 'on-demand' 인코딩 작업**을 지원하며, API 또는 콘솔을 통해 요청할 수 있습니다.
        

## 2. **Per-Title 인코딩 (개별 콘텐츠 최적화)**

- **지원 가능 여부**:
    
    - MediaConvert는 **Per-Title 인코딩 기능을 지원**하며, 이를 통해 영상별로 최적화된 인코딩 세팅을 자동으로 적용할 수 있습니다.
        
    - **자동 분석 후 최적의 인코딩 설정을 생성하는 기능**(예: 콘텐츠의 복잡도에 따른 비트레이트 조절, 해상도 조정 등)을 제공하며, 이 설정은 작업 템플릿이나 API를 통해 쉽게 적용 가능합니다.
        
    - **QVBR(품질 정의 가변 비트레이트)**와 같은 기술도 지원하여, 영상 특성에 맞게 최적화된 인코딩이 가능합니다.
        

## 결론

- **적용 가능 여부**:
    
    - **네, AWS MediaConvert는 두 방법 모두 지원**합니다.
        
    - **실시간 요청에 따른 즉시 인코딩(온디맨드)** 또는 **사전 인코딩 후 저장/캐싱**하는 방식 모두 활용할 수 있으며,
        
    - 특히 **Per-Title 인코딩은 콘텐츠별 최적화에 적합**하고, **실시간 트랜스코딩은 라이브 또는 긴급 요청에 적합**합니다.
        

## 참고

- MediaConvert는 **API 또는 콘솔을 통해 인코딩 작업을 요청하는 구조**이기 때문에, 필요에 따라 사전 인코딩 또는 실시간 인코딩을 선택할 수 있습니다.
    
- **자동화와 최적화를 위해** AWS Lambda, Step Functions 등과 연동해 인코딩 워크플로우를 구성하는 것도 가능합니다.