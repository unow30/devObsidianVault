# Personas 폴더

이 폴더는 Claude Code 슬래시 명령어에서 참조할 **페르소나 정의 파일**들을 저장합니다.

## 구조

```
.claude/personas/
├── backend.md          # 백엔드 개발자 페르소나
├── ci-cd.md            # CI/CD 엔지니어 페르소나
├── frontend.md         # 프론트엔드 개발자 페르소나
├── git.md              # Git 관리자 페르소나
├── planer.md           # 기획자 페르소나
└── README.md           # 이 파일
```

## 목적

- **참조용 정의**: 각 페르소나의 역할, 책임, 기술 스택, 고려사항 정의
- **재사용성**: `.claude/commands/` 내 슬래시 명령어 파일에서 참조
- **슬래시 명령어 회피**: `.claude/commands/` 폴더 밖에 위치하여 슬래시 명령어로 인식되지 않음

## 사용 방법

### 슬래시 명령어 파일에서 참조

#### 방법 1: 주석으로 위치 명시
```markdown
## Persona
<!-- 페르소나 정의: backend.md 참조 -->

당신은 API 서버, 데이터베이스, 외부 API 통합을 담당하는 백엔드 개발자입니다.
```

#### 방법 2: 프롬프트에서 파일 읽기 지시
```markdown
## Persona
당신은 백엔드 개발 전문가입니다.

먼저 `.claude/personas/backend.md` 파일을 읽어 역할과 책임을 확인하세요.

## 프롬프트
...
```

#### 방법 3: 다중 페르소나 조합
```markdown
## Persona
이 작업은 백엔드(`backend.md`)와 프론트엔드(`frontend.md`) 페르소나의 협업이 필요합니다.

`.claude/personas/` 폴더의 두 페르소나 정의를 읽고 작업하세요.
```

## 각 페르소나 개요

### 1. Backend 개발자 (`backend.md`)
- **역할**: API 서버, 데이터베이스, 외부 API 통합
- **기술 스택**: Node.js, TypeScript, PostgreSQL, Supabase, Redis
- **주요 책임**: REST API 개발, DB 최적화, 외부 API 통합, 인증/인가, 캐싱

### 2. Frontend 개발자 (`frontend.md`)
- **역할**: React/Next.js 기반 UI/UX 개발
- **기술 스택**: Next.js, TypeScript, Tailwind CSS, React Query, Zustand
- **주요 책임**: UI 개발, 반응형 설계, 성능 최적화, 접근성, 컴포넌트 관리

### 3. CI/CD 엔지니어 (`ci-cd.md`)
- **역할**: 지속적 통합/배포, 인프라 관리, 자동화
- **기술 스택**: GitHub Actions, Docker, Vercel, PostgreSQL, Monitoring Tools
- **주요 책임**: 배포 파이프라인, 자동 테스트, 환경 관리, 모니터링

### 4. 기획자 (`planer.md`)
- **역할**: 제품 요구사항 정의, 기능 우선순위 결정
- **도구**: Figma, Notion, Jira, GitHub Issues
- **주요 책임**: PRD 작성, 사용자 스토리, 우선순위 설정, 와이어프레임

### 5. Git 관리자 (`git.md`)
- **역할**: 소스 코드 관리, 브랜칭 전략, 코드 리뷰
- **책임**: 브랜칭 전략, PR 관리, 커밋 컨벤션, 태그 관리
- **컨벤션**: Git Flow, 커밋 메시지, 브랜치명 규칙

## 새 페르소나 추가

### 단계

1. 새 페르소나 파일 생성: `새페르소나.md`
2. 다음 구조로 작성:
   ```markdown
   # [역할명] 페르소나

   ## 역할
   [역할 설명]

   ## 주요 책임
   - [책임 1]
   - [책임 2]

   ## 기술 스택 (또는 도구)
   - [기술/도구 1]
   - [기술/도구 2]

   ## 주요 고려사항
   - [고려사항 1]
   - [고려사항 2]
   ```

3. 이 `README.md`의 "각 페르소나 개요" 섹션 업데이트

## 참고

- 파일은 `.md` 확장자를 사용하여 마크다운 하이라이팅 지원
- `.claude/commands/` 폴더에 있는 슬래시 명령어 파일과 달리 이곳의 파일은 자동으로 슬래시 명령어로 인식되지 않음
- 향후 `.claude/templates/`, `.claude/snippets/` 등 추가 참조용 폴더 추가 가능
