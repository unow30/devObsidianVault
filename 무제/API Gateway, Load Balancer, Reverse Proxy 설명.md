API Gateway, Load Balancer, Reverse Proxy는 모두 “클라이언트와 서버 사이에서 요청을 받아서 다른 서버로 보내는 중간 레이어”지만, 해결하려는 문제와 기능 범위가 다릅니다.

***

## Reverse Proxy

Reverse Proxy는 서버 앞에 앉아서 “대신 받아주고 대신 보내주는” 일반적인 프록시입니다.

주요 역할  
- 클라이언트는 실제 애플리케이션 서버를 직접 보지 않고, 프록시에게만 요청을 보냄.  
- 프록시는 적절한 백엔드 서비스로 요청을 포워딩하고, 응답을 받아 클라이언트에 전달.  

대표 기능  
- SSL 종료(HTTPS Termination): TLS 복호화를 프록시에서 처리해서, 뒤쪽 서버는 HTTP로 단순하게 동작하게 함.  
- 캐싱: 자주 요청되는 이미지·정적 파일·API 응답 등을 캐시해서 백엔드 부하 감소.  
- 보안: 실제 서버 IP를 감추고, WAF·IP 차단 등 보안 정책을 프록시 레벨에서 적용.  
- 압축·헤더 조작: gzip 압축, 공통 헤더 추가 등 트래픽 최적화.  

툴 예시: Nginx, HAProxy, Caddy, Apache를 “앞단에 세우면” 전형적인 Reverse Proxy 패턴입니다.

***

## Load Balancer

Load Balancer는 “여러 개의 서버 인스턴스 사이로 트래픽을 나눠주는 데 특화된 Reverse Proxy”입니다.

핵심 목적  
- Scalability: 서버 인스턴스를 여러 개 두고, 요청을 분산해 더 많은 트래픽을 처리.  
- Availability: 한 서버가 죽으면 헬스 체크로 감지하고, 살아있는 서버로만 트래픽을 보내 서비스 중단을 피함.  

분산 알고리즘 예  
- Round-robin: 순번대로 A → B → C → 다시 A.  
- Least connections: 현재 처리 중인 연결이 가장 적은 서버로 보냄.  
- IP hash: 특정 클라이언트 IP는 항상 같은 서버로 보내 “세션 스티키”를 유지.  
- Weighted: 스펙 좋은 서버에 더 많은 트래픽을 주는 가중치 분산.  

L4 vs L7  
- L4 Load Balancer: TCP/UDP 레벨에서 IP·포트만 보고 라우팅, 빠르지만 HTTP 컨텍스트는 모름.  
- L7 Load Balancer: HTTP 레벨에서 Path, Header, Cookie를 보고 라우팅, `/api/users`는 A 풀, `/api/orders`는 B 풀 등에 보낼 수 있음.  

예: AWS ALB/NLB, GCP Load Balancer, Nginx/HAProxy의 LB 모드 등.

***

## API Gateway

API Gateway는 “API를 외부에 노출할 때 생기는 공통 문제(인증, 요금제, 버저닝 등)를 한 곳에서 처리하는 API 특화 Reverse Proxy”입니다.

주요 책임  
- 인증·인가: 토큰 검증(JWT 등), 권한 체크를 게이트웨이에서 통합 처리해서, 내부 마이크로서비스는 비즈니스 로직에만 집중.  
- Rate limiting: 무료 플랜은 분당 100 요청, 유료는 1,000 요청처럼 클라이언트별·플랜별 사용량 제한을 걸어 백엔드 보호.  
- 요청/응답 변환: 모바일 앱은 JSON, 레거시 서비스는 XML을 쓴다면, 게이트웨이가 포맷을 변환.  
- API 버저닝·라우팅: `/v1/users`는 구버전 서비스, `/v2/users`는 신버전 서비스로 라우팅해서 점진적 마이그레이션.  
- 로깅·모니터링: 어떤 클라이언트가 어떤 엔드포인트를 얼마나, 어느 latency로 호출했는지 중앙에서 수집.  

예: Kong, AWS API Gateway, Apigee, Azure API Management, Tyk 등.

***

## 왜 헷갈리는가? (경계/스펙트럼)

실제 제품들은 기능이 겹치기 때문입니다.

- Nginx: 기본은 Reverse Proxy지만, 로드 밸런싱 기능과 Rate limiting, 간단한 인증까지 지원 → 일부 API Gateway 역할까지 가능.  
- Kong: Nginx 위에 구축된 API Gateway라서, 내부적으로 Reverse Proxy·Load Balancer 기능을 그대로 사용.  
- 클라우드: AWS는 ALB(Load Balancer)와 API Gateway를 별도 상품으로 팔지만, ALB도 L7 라우팅을 제공하고 API Gateway도 내부적으로 트래픽 분산을 함.  

영상에서 설명하듯, 이 셋은 “완전히 다른 세 박스”라기보다는 아래와 같은 **스펙트럼**으로 보는 게 좋습니다.  

- 왼쪽: 단순 포워딩·SSL 종료·캐싱 → Reverse Proxy.  
- 중간: 여기에 트래픽 분산·헬스 체크·Failover → Load Balancer.  
- 오른쪽: 여기에 인증·요금제·Rate limit·변환·분석 → API Gateway.  

***

## 언제 무엇을 쓸까? (실전 감각)

영상에서 제시한 간단한 의사결정 프레임워크를 정리하면 다음과 같습니다.

| 상황/요구사항 | 적합한 컴포넌트 | 이유 |
| --- | --- | --- |
| 단일/소수 서버, SSL 종료·캐싱만 필요 | Reverse Proxy(Nginx 등) | 앞단에서 HTTPS 처리, 캐싱, 간단한 라우팅만 있으면 충분. |
| 동일 서비스를 여러 인스턴스로 수평 확장 | Load Balancer | 트래픽 분산·헬스 체크·Failover가 핵심. |
| 외부 개발자용 공용 API, 플랜별 요금제·키 발급 | API Gateway | 인증·Rate limiting·키 관리·분석 등 API 관리 기능 필요. |
| 마이크로서비스 다수, 공통 Cross-cutting concern 정리 | API Gateway + 내부 Reverse Proxy/LB | 인증·로그·Rate limit은 게이트웨이, 서비스별 Nginx는 정적자원·SSL 담당. |

전형적인 구성 예  
- 클라이언트 → CDN(전 세계 엣지 Reverse Proxy, 정적 캐시) → API Gateway(인증, 요금제, 라우팅) → Load Balancer(서비스 인스턴스들로 분산) → 각 서비스 앞 Nginx(내부 SSL·정적 파일).  

요약하면,
![[Pasted image 20260211152022.png]]
- Reverse Proxy: “앞에서 대신 받아주는 일반 프록시 레이어”.  
- Load Balancer: “여러 서버로 나눠주는 스케일·가용성 레이어”.  
- API Gateway: “API 노출·보안·요금제·분석을 담당하는 API 관리 레이어”.

출처
https://www.youtube.com/watch?v=DVR0zvDYJgY

