Vercel의 React와 Next.js 성능 최적화 가이드라인을 담은 스킬 문서다. 8개 카테고리에 45개 규칙을 우선순위별로 정리한 내용으로, 코드 작성·리뷰·리팩토링 시 활용하도록 설계되었다.[1]

## 스킬 메타데이터

```
---
name: vercel-react-best-practices
description: Vercel Engineering의 React 및 Next.js 성능 최적화 가이드라인. React/Next.js 코드 작성, 리뷰, 리팩토링 시 최적 패턴을 보장하기 위해 사용. React 컴포넌트, Next.js 페이지, 데이터 페칭, 번들 최적화, 성능 개선 작업에서 트리거.
license: MIT
meta
  author: vercel
  version: "1.0.0"
---
```
이 스킬은 React/Next.js 관련 작업에서 자동으로 제안되도록 트리거된다.[1]

## 적용 시기

이 가이드라인을 참조할 때:
- 새로운 React 컴포넌트나 Next.js 페이지 작성 시
- 데이터 페칭(클라이언트 또는 서버 사이드) 구현 시
- 성능 문제 코드 리뷰 시
- 기존 React/Next.js 코드 리팩토링 시
- 번들 크기나 로드 시간 최적화 시[1]

## 규칙 카테고리 (우선순위별)

| 우선순위 | 카테고리              | 영향도     | 접두사     |
|----------|-----------------------|------------|------------|
| 1        | 워터폴 제거           | CRITICAL  | `async-`  |
| 2        | 번들 크기 최적화      | CRITICAL  | `bundle-` |
| 3        | 서버 사이드 성능      | HIGH      | `server-` |
| 4        | 클라이언트 데이터 페칭 | MEDIUM-HIGH | `client-` |
| 5        | 리렌더 최적화         | MEDIUM    | `rerender-` |
| 6        | 렌더링 성능           | MEDIUM    | `rendering-` |
| 7        | JavaScript 성능       | LOW-MEDIUM| `js-`     |
| 8        | 고급 패턴             | LOW       | `advanced-` |[1]

## 빠른 참조 (Quick Reference)

### 1. 워터폴 제거 (CRITICAL)
- `async-defer-await` - 실제 사용되는 브랜치로 await 이동
- `async-parallel` - 독립 작업에 Promise.all() 사용
- `async-dependencies` - 부분 의존성에 better-all 사용
- `async-api-routes` - API 라우트에서 프로미스 일찍 시작, 늦게 await
- `async-suspense-boundaries` - 콘텐츠 스트리밍에 Suspense 사용[1]

### 2. 번들 크기 최적화 (CRITICAL)
- `bundle-barrel-imports` - 배럴 파일 피하고 직접 임포트
- `bundle-dynamic-imports` - 무거운 컴포넌트에 next/dynamic 사용
- `bundle-defer-third-party` - hydration 후 분석/로깅 로드
- `bundle-conditional` - 기능 활성화 시에만 모듈 로드
- `bundle-preload` - 호버/포커스 시 프리로딩으로 지각 속도 향상[1]

### 3. 서버 사이드 성능 (HIGH)
- `server-cache-react` - 요청당 중복 제거에 React.cache()
- `server-cache-lru` - 요청 간 캐싱에 LRU 캐시
- `server-serialization` - 클라이언트 컴포넌트로 전달 데이터 최소화
- `server-parallel-fetching` - 페칭 병렬화 위해 컴포넌트 재구조화
- `server-after-nonblocking` - 비동기 작업에 after() 사용[1]

### 4. 클라이언트 사이드 데이터 페칭 (MEDIUM-HIGH)
- `client-swr-dedup` - 자동 요청 중복 제거에 SWR 사용
- `client-event-listeners` - 전역 이벤트 리스너 중복 제거[1]

### 5. 리렌더 최적화 (MEDIUM)
- `rerender-defer-reads` - 콜백에서만 사용하는 상태는 구독 피함
- `rerender-memo` - 비싼 작업을 메모이즈 컴포넌트로 추출
- `rerender-dependencies` - effect에서 기본형 의존성 사용
- `rerender-derived-state` - 원시 값이 아닌 파생 불리언 구독
- `rerender-functional-setstate` - 안정 콜백에 함수형 setState
- `rerender-lazy-state-init` - 비싼 값에 useState 함수 전달
- `rerender-transitions` - 비긴급 업데이트에 startTransition[1]

### 6. 렌더링 성능 (MEDIUM)
- `rendering-animate-svg-wrapper` - SVG가 아닌 div 래퍼 애니메이션
- `rendering-content-visibility` - 긴 리스트에 content-visibility
- `rendering-hoist-jsx` - 정적 JSX를 컴포넌트 밖으로 추출
- `rendering-svg-precision` - SVG 좌표 정밀도 줄임
- `rendering-hydration-no-flicker` - 클라이언트 전용 데이터에 인라인 스크립트
- `rendering-activity` - show/hide에 Activity 컴포넌트
- `rendering-conditional-render` - 조건부에 && 대신 삼항 연산자[1]

### 7. JavaScript 성능 (LOW-MEDIUM)
- `js-batch-dom-css` - 클래스나 cssText로 CSS 변경 그룹화
- `js-index-maps` - 반복 조회에 Map 빌드
- `js-cache-property-access` - 루프에서 객체 속성 캐싱
- `js-cache-function-results` - 모듈 레벨 Map에 함수 결과 캐싱
- `js-cache-storage` - localStorage/sessionStorage 읽기 캐싱
- `js-combine-iterations` - 여러 filter/map을 하나의 루프로 결합
- `js-length-check-first` - 비싼 비교 전에 배열 길이 체크
- `js-early-exit` - 함수에서 조기 반환
- `js-hoist-regexp` - 루프 밖으로 RegExp 생성 호이스팅
- `js-min-max-loop` - sort 대신 루프로 min/max
- `js-set-map-lookups` - O(1) 조회에 Set/Map
- `js-tosorted-immutable` - 불변성에 toSorted() 사용[1]

### 8. 고급 패턴 (LOW)
- `advanced-event-handler-refs` - 이벤트 핸들러를 refs에 저장
- `advanced-use-latest` - 안정 콜백 refs에 useLatest[1]

## 사용 방법

개별 규칙 파일을 읽어 상세 설명과 코드 예시 확인:
```
rules/async-parallel.md
rules/bundle-barrel-imports.md
rules/_sections.md
```
각 규칙 파일에는 왜 중요한지, 잘못된 코드 예시, 올바른 코드 예시, 추가 맥락이 포함되어 있다.[1]

## 전체 컴파일 문서
모든 규칙 확장된 전체 가이드는 `AGENTS.md`에서 확인.[1]

출처
[1] SKILL.md https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/55485747/f015c447-96c9-4896-803a-2eddcc16f8dc/SKILL.md
