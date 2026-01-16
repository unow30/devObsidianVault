## 목표
- 로컬 Supabase 인스턴스로 개발
- GitHub Actions를 통해서만 프로덕션 Supabase 접근
- Vercel 배포 연동

***

## 구현 단계

### 1단계: Supabase CLI 설치 및 초기화
```bash
# Supabase CLI 설치 (macOS)
brew install supabase/tap/supabase

# 프로젝트 초기화
supabase init
```

**생성되는 구조:**
```
supabase/
├── config.toml
├── migrations/
└── seed.sql
```

**package.json 수정 (스크립트 추가):**
```json
{
  "scripts": {
    "supabase:start": "supabase start",
    "supabase:stop": "supabase stop",
    "supabase:reset": "supabase db reset",
    "supabase:gen:types": "supabase gen types typescript --local > types/database.types.ts"
  }
}
```

***

### 2단계: 환경 변수 구조 정리

**파일 구조:**
```
.env.local.example    # Git 추적 (템플릿)
.env.local            # Git 무시 (로컬 개발)
```

**.env.local.example 내용:**
```env
# 로컬 Supabase (supabase start 후 출력되는 값 사용)
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_local_anon_key
```

**.gitignore 추가:**
```
# Supabase
supabase/.branches
supabase/.temp
.supabase
```

***

### 3단계: 마이그레이션 파일 생성
**파일:** `supabase/migrations/00000000000000_initial_schema.sql`

- cities 테이블 (PRD 요구사항 기반)
- RLS 정책 설정
- 인덱스 생성

***

### 4단계: 시드 데이터 작성
**파일:** `supabase/seed.sql`

- 15개 도시 초기 데이터 (PRD 요구사항)
- 개발/테스트용 데이터

***

### 5단계: GitHub Actions 워크플로

**파일 1:** `.github/workflows/ci.yml`
- PR 시 실행
- Lint, TypeScript 체크
- 마이그레이션 테스트 (로컬 Supabase)
- 빌드 테스트

**파일 2:** `.github/workflows/deploy-migrations.yml`
- main 브랜치 push 시 실행
- 프로덕션 Supabase에 마이그레이션 배포
- `supabase db push` 실행

**필요한 GitHub Secrets:**

| Secret                  | 설명                    |
|-------------------------|-------------------------|
| `SUPABASE_ACCESS_TOKEN` | Supabase 개인 액세스 토큰 |
| `PRODUCTION_PROJECT_ID` | exkmrmhkwsoekuzhtgtl    |
| `PRODUCTION_DB_PASSWORD`| 프로덕션 DB 비밀번호     |

***

### 6단계: Vercel 환경 변수 설정

**Vercel Dashboard에서 설정:**

| 변수                       | Production/Preview |
|----------------------------|-------------------|
| `NEXT_PUBLIC_SUPABASE_URL` | 프로덕션 URL      |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | 프로덕션 anon key |

***

### 7단계: 개발자 온보딩 스크립트
**파일:** `scripts/setup.sh`

- 필수 도구 확인 (Node, Docker, Supabase CLI)
- 패키지 설치
- 환경 변수 파일 생성
- Supabase 로컬 시작
- DB 초기화

***

## 생성/수정 파일 목록

| 파일                              | 작업                  |
|-----------------------------------|-----------------------|
| `supabase/config.toml`            | 생성 (supabase init)  |
| `supabase/migrations/*.sql`       | 생성                  |
| `supabase/seed.sql`               | 생성                  |
| `.github/workflows/ci.yml`        | 생성                  |
| `.github/workflows/deploy-migrations.yml` | 생성       |
| `scripts/setup.sh`                | 생성                  |
| `types/database.types.ts`         | 생성                  |
| `package.json`                    | 수정 (스크립트 추가)  |
| `.env.local.example`              | 수정                  |
| `.gitignore`                      | 수정                  |
| `.env`                            | 삭제 (프로덕션 키 노출 방지) |

***

## 워크플로 다이어그램

```
[로컬 개발]        [GitHub PR]         [main 머지]        [Production]
     │               │                   │                  │
     │ supabase start│                   │                  │
     │ npm run dev   │                   │                  │
     ├────git push───►│                   │                  │
     │               │ CI 워크플로 실행   │                  │
     │               │ - lint/typecheck  │                  │
     │               │ - migration test  │                  │
     │               │ - build test      │                  │
     │               ├────PR 머지────────►│                  │
     │               │                   │ deploy-migrations│
     │               │                   ├──────────────────►│
     │               │                   │ supabase db push │
     │               │                   │                  │
     │               │                   │ Vercel 자동 배포  │
     │               │                   ├──────────────────►│
```

***

## 검증 방법

### 로컬 개발 환경 테스트
```bash
# 1. 설정 스크립트 실행
./scripts/setup.sh

# 2. 개발 서버 시작
npm run dev

# 3. Supabase Studio 확인
# http://127.0.0.1:54323

# 4. 로그인/회원가입 테스트
# http://localhost:3000/login
```

### CI/CD 테스트
1. 새 브랜치에서 마이그레이션 파일 추가
2. PR 생성 → CI 워크플로 확인
3. main 머지 → 마이그레이션 배포 워크플로 확인
4. Vercel 배포 확인

### Inbucket (이메일 테스트)
- `http://127.0.0.1:54324`
- 회원가입 시 확인 이메일 테스트

출처
