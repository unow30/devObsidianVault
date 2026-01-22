markdown

````
# 🚀 Turborepo Full-Stack Project Plan
(Next.js + Nest.js + pnpm Workspaces)

이 문서는 Turborepo를 기반으로 한 프론트엔드(Next.js), 백엔드(Nest.js), 그리고 공통 패키지 구조를 구축하기 위한 마스터 플랜입니다.

## 1. 프로젝트 구조 (Project Layout)
```text
my-monorepo/
├── apps/
│   ├── web/          # Next.js (Frontend)
│   └── api/          # Nest.js (Backend)
├── packages/
│   ├── common/       # DTO, Types, Utilities (공유 패키지)
│   ├── tsconfig/     # 공통 TypeScript 설정
│   └── eslint-config/ # 공통 린트 설정
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
````

코드를 사용할 때는 주의가 필요합니다.

2. 초기화 단계 (Setup Phase)

2.1 워크스페이스 활성화

`pnpm-workspace.yaml` 파일을 생성하여 패키지 경로를 지정합니다.  
pnpm Workspaces 가이드

2.2 파이프라인 설정

`turbo.json`에 빌드 및 개발 워크플로우를 정의합니다.

json

```
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": { "cache": false, "persistent": true }
  }
}
```

코드를 사용할 때는 주의가 필요합니다.

3. 핵심 공유 로직 (Shared Logic)

3.1 공통 DTO 공유

`@repo/common` 패키지에서 `class-validator`를 사용하여 프론트/백 공통 타입을 관리합니다.

- **Next.js**: React Hook Form과 연동하여 클라이언트 검증.
- **Nest.js**: ValidationPipe와 연동하여 서버 보안 강화.

3.2 TypeScript Base Config

`packages/tsconfig/base.json`을 통해 Nest.js에 필요한 데코레이터 설정(`emitDecoratorMetadata`)을 전역 관리합니다.

4. 배포 전략 (Deployment)

4.1 Docker 최적화

`turbo prune`을 사용하여 이미지 크기를 최소화합니다.

1. `pruner` 단계: 필요한 소스만 추출.
2. `installer` 단계: 의존성 설치 및 빌드.
3. `runner` 단계: 최소 실행 파일만 포함 (Next.js Standalone 모드 활용).

4. 실행 명령어 (Scripts)

- 모든 앱 설치: `pnpm install`
- 전체 실행: `pnpm turbo dev`
- 특정 앱 빌드: `pnpm turbo build --filter=@repo/api`

````

---

### 🛠️ 바로 실행 가능한 명령어
아래 명령어를 터미널에 입력하면 Turborepo에서 공식 제공하는 기본 보일러플레이트를 즉시 생성할 수 있습니다.

```bash
# Turborepo 공식 스타터 생성 (가장 권장되는 시작 방식)
npx create-turbo@latest
````

**Next.js와 Nest.js를 연결할 때 발생하는 CORS 문제나 API 연동 코드(Axios/Fetch) 설정**까지 추가로 도와드릴까요? 원하신다면 **공통 API 클라이언트 패키지** 구성법을 안내해 드릴 수 있습니다.