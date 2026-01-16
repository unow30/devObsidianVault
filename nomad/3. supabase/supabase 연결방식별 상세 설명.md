### 1. Direct Connection (직접 연결)

- **가장 전통적인 방식**입니다. 클라이언트(DBeaver)가 Supabase PostgreSQL 인스턴스에 직접 연결하여 하나의 전용 연결을 오랫동안 사용합니다.
    
- **장점:** 모든 PostgreSQL 기능을 완벽하게 지원하며, 세션 기반 설정(`SET` 명령어 등)을 유지하는 데 문제가 없습니다. DBeaver에서 데이터베이스를 탐색하고 관리하는 데 가장 익숙하고 안정적인 방식입니다.
    
- **단점:** 데이터베이스가 수용할 수 있는 **최대 연결 수(`max_connections`)**가 제한적이기 때문에, 많은 수의 동시 사용자 요청이 발생하면 연결 제한에 쉽게 도달할 수 있습니다.
    
- **참고:** Supabase에서 Direct Connection은 기본적으로 **IPv6**를 권장합니다.
    

### 2. Session Pooler (세션 풀러)

- **연결 풀링(Connection Pooling)** 기술을 사용하지만, 동작 방식은 Direct Connection과 유사합니다.
    
- 클라이언트가 접속하면 풀러가 데이터베이스 연결을 **할당하고 클라이언트가 끊을 때까지 유지**합니다. 즉, 여러 클라이언트가 직접 연결 수 제한을 초과하지 않도록 **큐(Queue)와 연결 공유 기능**을 제공하지만, 연결을 '오래' 유지한다는 점은 Direct Connection과 같습니다.
    
- **장점:** Direct Connection의 대안으로 적합하며, 특히 **IPv4 환경**에서 사용하기 좋습니다. DBeaver와 같이 **세션 상태 유지가 필요한** GUI 도구에 매우 적합합니다.
    
- **단점:** Direct Connection처럼 연결을 오래 유지하므로, 서버리스 환경과 같이 연결 생성 및 해제가 잦은 환경에서는 비효율적일 수 있습니다.
    

### 3. Transaction Pooler (트랜잭션 풀러)

- **연결 풀링**의 장점을 극대화한 방식으로, **가장 높은 확장성**을 제공합니다.
    
- 클라이언트가 SQL 쿼리를 실행할 때 필요한 **트랜잭션이 진행되는 동안에만** 데이터베이스 연결을 할당하고, 트랜잭션이 완료되는 즉시 연결을 풀에 반환하여 다른 클라이언트가 즉시 재사용할 수 있도록 합니다.
    
- **장점:** 적은 수의 실제 데이터베이스 연결로 **수백 배 이상의 클라이언트 동시 접속**을 처리할 수 있어, 서버리스 환경(Edge/Serverless Functions)처럼 짧고 폭발적인 연결이 필요한 경우에 최적입니다.
    
- **단점:** 트랜잭션 단위로 연결이 끊어지고 재사용되므로, **세션 기반 상태(Session-level state)**를 유지할 수 없습니다. 따라서 DBeaver와 같은 GUI 툴에서 사용하기에는 불편할 수 있습니다. (예: `PREPARE STATEMENT` 사용 불가)

---
### 4. Supabase에서 "no route to host" 오류
Supabase에서 "no route to host" 오류는 주로 직접 연결(Direct Connection, 포트 5432) 시 IPv6 네트워크 지원 부족으로 발생합니다.[1][2][3] 한국의 많은 가정 인터넷(예: ipTIME 공유기)에서 IPv6가 제대로 지원되지 않아 문제가 흔합니다.[4]

ipv4를 이용해서 접근하기 위해선 전용 ipv4추과 기능을 활성화해야 합니다.
![[Pasted image 20260110111200.png]]
#### 주요 원인
Supabase DB 호스트(db.xxx.supabase.co)는 IPv6 전용으로 동작하며, IPv4 연결이 불가능합니다.[3][2] 클라이언트(예: DBeaver)가 IPv6을 지원하지 않거나 네트워크가 차단되면 연결 실패합니다.[4][5]

#### 해결 방법
- **Connection Pooler 사용 (권장)**: Supabase 대시보드 > Settings > Database > Connection Pooler > Session 또는 Transaction 모드(포트 6543)의 호스트와 문자열 사용. IPv4 호환됩니다.[3][6][7]
  - Host: aws-0-xxx.pooler.supabase.com (IPv4 지원)
  - ?pgbouncer=true 파라미터 추가 (구버전 경우).[8]
- **IPv6 테스트 및 활성화**:
  ```
  ping6 db.xxx.supabase.co
  nc -zv db.xxx.supabase.co 5432
  ```
  IPv6 미지원 시 Cloudflare WARP 클라이언트 설치로 우회.[4]

#### 추가 확인
- 네트워크 방화벽/VPN 확인: 한국 ISP나 회사망에서 차단 가능.[9]
- DBeaver: macOS/Windows 로컬 네트워크 권한 허용 (Privacy & Security).[5]
- IP 밴 확인: Supabase 대시보드 > Network Restrictions에서 IP 언밴.[10]

##### 출처
[1] NoRouteToHost connection issue https://www.reddit.com/r/Supabase/comments/1l27307/noroutetohost_connection_issue/
[2] no route to host - Supabase https://www.answeroverflow.com/m/1394688155323596820
[3] Connect to your database | Supabase Docs https://supabase.com/docs/guides/database/connecting-to-postgres
[4] Supabase PostgreSQL 연결 - Direct Connect 트러블슈팅 https://physickskim.tistory.com/entry/Supabase-PostgreSQL-%EC%97%B0%EA%B2%B0-Direct-Connect-%ED%8A%B8%EB%9F%AC%EB%B8%94%EC%8A%88%ED%8C%85
[5] "no route to host" in macOS Sequoia (and a workaround) #35705 https://github.com/dbeaver/dbeaver/issues/35705
[6] Supabase Connection Pooling Guide https://store-restack.vercel.app/docs/supabase-knowledge-supabase-connection-pooling
[7] Supavisor FAQ · supabase · Discussion #21566 https://github.com/orgs/supabase/discussions/21566
[8] Supabase Connection String for IPv6 in Prisma - SaasRock https://saasrock.com/docs/articles/supabase-connection-string-for-ipv6-in-prisma
[9] DBeaver와 supabase connection - Inflearn | Community Q&A https://www.inflearn.com/en/community/questions/1698598/dbeaver%EC%99%80-supabase-connection
[10] Can't link project (connect: no route to host) · Issue #1996 · supabase/cli https://github.com/supabase/cli/issues/1996
[11] Confused about ipv6 and supabase link https://www.reddit.com/r/Supabase/comments/1ay29g9/confused_about_ipv6_and_supabase_link/
[12] supabase link command failed: no route to host - Answer Overflow https://www.answeroverflow.com/m/1442763361484935308
[13] Supabase Local Connection String: Setup & Troubleshooting https://spaceworx.net/blog/supabase-local-connection-string-setup
[14] supabase postgres accepting TCP/IP connections error https://www.reddit.com/r/Supabase/comments/1ii85pe/supabase_postgres_accepting_tcpip_connections/
[15] Route not found when in production · supabase · Discussion #368 https://github.com/orgs/supabase/discussions/368
[16] failed to connect to postgres: failed to connect to `host=db.[id].supabase.co user=postgres database=postgres`: dial error (dial tcp [2406:da14:271:9903:c1bf:5f4c:ff8a:9e0c]:5432: connect: no route to host) · Issue #3318 · supabase/cli https://github.com/supabase/cli/issues/3318
[17] Thread: psql: could not connect to server: No route to host https://postgrespro.com/list/thread-id/1160434
[18] I get error when connecting local supabase instance to a remote project https://www.reddit.com/r/Supabase/comments/1bo3wuv/i_get_error_when_connecting_local_supabase/
[19] Database Connections https://docs-rog1zs1kv-supabase.vercel.app/docs/guides/database/connecting-to-postgres
[20] dial error (dail tcp) · Issue #21143 · supabase/supabase https://github.com/supabase/supabase/issues/21143
[21] Testing Supabase API routes with Postman https://www.reddit.com/r/Supabase/comments/10u0ctp/testing_supabase_api_routes_with_postman/
[22] Supabase & Your Network: IPv4 and IPv6 compatibility https://supabase.com/docs/guides/troubleshooting/supabase--your-network-ipv4-and-ipv6-compatibility-cHe3BP
[23] Can't connect to ipv6 database #33925 - supabase - GitHub https://github.com/orgs/supabase/discussions/33925
[24] Pooled vs direct connection https://www.reddit.com/r/Supabase/comments/1g164c2/pooled_vs_direct_connection/
[25] Setup fails at Supabase database step - IPv6 connection ... https://github.com/kortix-ai/suna/issues/1406
[26] DBeaver와 supabase connection - Inflearn | コミュニティ Q&A https://www.inflearn.com/ja/community/questions/1698598/dbeaver%EC%99%80-supabase-connection
[27] Docs unclear on connection pool vs. direct connection - Drizzle Team https://www.answeroverflow.com/m/1153690802095136809
