태블릿형 매장 결제 키오스크 솔루션을 위한 **포괄적인 PRD(Product Requirement Document)**와 **상세한 실행 계획(Execution Plan)**을 작성했습니다.

## 📋 문서 구성

**Part 1. PRD (제품 요구사항서)**
- 프로젝트명: kiosk1
- 목표 및 성공 지표: 매장에서 사용 가능한 태블릿용 결제 키오스크 만들기
- 사용자 스토리 (매장 직원, 고객, 매장주, 관리자)
	- 고객: 키오스크로 재화나 서비스를 선택하여 주문, 결제할 수 있다.
	- 매장 직원: 키오스크의 주문내역을 확인하고 재화나 서비스를 제공한다.
	- 매장주: 관리자 페이지에서 재화나 서비스 정보를 등록할 수 있다. 주문내역, 결제내역을 확인할 수 있다. 다양한 매장 활동에 대한 통계를 확인 및 출력할 수 있다.
	- 관리자: 여러 매장의 영업정보를 확인할 수 있다. 새로운 매장과 키오스크를 등록할 수 있다.
- 디자인 원칙 및 사용자 흐름
	- 여러 키오스크 서비스 운영 사이트 참고
- MVP 핵심 기능 및 부가 기능
	- 키오스크앱: 재화 및 서비스 선택하기, 주문하기, 결제하기
	- 관리자 페이지: 재화 및 서비스 관리하기, 주문내역 관리하기, 결제내역 관리하기, 정보 출력하기, 여러 매장정보 관리하기
- 기술 요구사항 (기본 + 추가 스택)
	- nestjs, nextjs, postgres, redis, toss payment, docker
- 비기능 요구사항 (성능, 보안, 가용성)
	- 여러 키오스크의 동시 주문에도 서버가 안정적으로 실행되야한다.
	- 결제정보가 유출되어선 안된다.
	- 직원과 관리자 이외의 사람이 키오스크를 종료해선 안된다.
	- 그밖에 필요한 기능 요구사항이 있다면 기술

**Part 2. 실행 계획 (기술 기반)**
- 목표 및 아키텍처 개요
- 상세 기술 스택 (Frontend, Backend, Infrastructure)
- Database Schema 및 API 엔드포인트
- 태스크 분해 (8가지 주요 작업)
  - **3.1 Backend**: Auth, Shop Management, Order, Payment, Analytics, Optimization, Error Handling
  - **3.2 Frontend (Kiosk UI)**: 프로젝트 설정, UI 라이브러리, 메뉴, 장바구니, 결제, 주문 완료
  - **3.3 Admin Panel**: 대시보드, 주문 관리, 메뉴 관리
  - **3.4 DevOps**: Docker, CI/CD
  - **3.5 테스트**: 단위 테스트, E2E 테스트

## 🎯 기술 스택 (요청사항 포함)

**기본 요구 기술**:
- Backend: **NestJS**
- Frontend: **Next.js**
- Database: **PostgreSQL**
- Cache: **Redis**

**추가 기술** (결제 키오스크 특성):
- WebSocket: **Socket.io** (실시간 주문 상태)
- ORM: **Prisma** (타입 안정성)
- Payment Gateway: **이니시스/KCP API**
- Storage: **AWS S3** (이미지 저장)
- DevOps: **Docker + Kubernetes + GitHub Actions**
- Monitoring: **Prometheus + Grafana**
- Logging: **ELK Stack**

## 📊 마일스톤

총 **10주** 내 MVP 출시:
- **M0 (1주)**: 준비 단계
- **M1 (2주)**: Backend MVP
- **M2 (2주)**: Frontend MVP
- **M3 (1주)**: Admin Panel
- **M4 (1주)**: 통합 & 테스트
- **M5 (1주)**: 배포 준비
- **M6 (1주)**: 출시

## 🔒 보안 & 성능

- **PCI-DSS 3.2.1** 준수
- **99.9% 가용성** SLA
- **1,000 TPS** 처리량
- **p95 < 500ms** API 응답

이 문서는 개발팀이 **즉시 실행 가능한 수준의 상세함**과 **전략적 방향성**을 모두 제공합니다.
```

이제 이 문서를 PDF로 다운로드하거나 필요시 특정 섹션을 추가로 보완할 수 있습니다!

출처
[1] 1.-project-requirement-document.md https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/55485747/b3d5eb61-449c-4c2f-9577-1b8eab0450bc/1.-project-requirement-document.md
[2] 2.-execution-plan.md https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/55485747/1bc1c609-0cbb-490f-b6b0-4afa915320ed/2.-execution-plan.md
[3] prdwa-silhaenggyehoegyi-caijeom.md https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/55485747/00e85feb-8d03-4c93-892b-6b1765fc6b00/prdwa-silhaenggyehoegyi-caijeom.md
```

---
schema

main_corner
sub_corner
item
item_option
item_image

recipt
영수증에 필수로 들어가야할 내용

---
https://brunch.co.kr/@fbrudtjr1/77
https://www.qvoss.co.kr/2024_kfc
https://yozm.wishket.com/magazine/detail/3552/