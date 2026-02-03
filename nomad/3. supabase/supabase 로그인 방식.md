## JWT 토큰 생성 과정

Supabase의 `signInWithPassword` 호출 시 로그인 성공하면 Access Token (JWT, 1시간 유효)과 Refresh Token이 자동 발급됩니다. `@supabase/ssr` 라이브러리가 이 토큰들을 HttpOnly, Secure 쿠키로 브라우저에 저장합니다.

## 상세 단계

1. **로그인 실행**: `app/actions/auth.ts`에서 `supabase.auth.signInWithPassword(data)` 호출.
2. **서버 응답**: Supabase가 JWT Access Token과 Refresh Token 발급.
3. **자동 쿠키 저장**: `utils/supabase/server.ts`의 `setAll(cookiesToSet)`에서 `cookieStore.set(name, value, options)` 실행.
4. **생성 쿠키**: `sb-{project-ref}-auth-token` (JWT), `sb-{project-ref}-auth-token-code-verifier` 등. 속성: HttpOnly=true, Secure=true, SameSite=Lax.

## 흐름 다이어그램

```
signInWithPassword() 호출
      ↓
Supabase 서버 인증
      ↓
JWT + Refresh Token 발급 [web:2]
      ↓
@supabase/ssr setAll() 호출 [web:16]
      ↓
cookieStore.set()으로 쿠키 저장 [web:8]
      ↓
브라우저 쿠키 생성 완료 ✓
```

## 핵심 포인트

개발자는 쿠키 설정 코드를 직접 작성할 필요 없이 `@supabase/ssr`가 전체 과정을 자동 처리합니다. Next.js App Router에서 middleware가 세션 유지와 토큰 갱신을 보장합니다. 이는 보안상 JavaScript 접근 불가능한 HttpOnly 쿠키 덕분입니다.



## config.toml
supabase/config.toml에 `[auth]` 관련 설정이 가능하다.
```toml
# How long tokens are valid for, in seconds. Defaults to 3600 (1 hour), maximum 604,800 (1 week).  
jwt_expiry = 3600

# If disabled, the refresh token will never expire.  
enable_refresh_token_rotation = true

# Allows refresh tokens to be reused after expiry, up to the specified interval in seconds.  
Requires enable_refresh_token_rotation = true
refresh_token_reuse_interval = 10
...
```
