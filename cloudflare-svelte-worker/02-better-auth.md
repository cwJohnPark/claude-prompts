---
name: better-auth
description: Better Auth를 활용한 SvelteKit 인증 구현 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, better-auth, authentication, d1]
---

# Better Auth

### 설치 및 기본 설정

```bash
bun add better-auth
```

### 서버 설정
```typescript
// src/lib/server/auth.ts
import { betterAuth } from 'better-auth';
import { D1Dialect } from 'kysely';

/**
 * Better Auth 인스턴스 생성
 * @param db - Cloudflare D1 데이터베이스
 */
export function createAuth(db: D1Database) {
  return betterAuth({
    // D1 데이터베이스 연결
    database: {
      dialect: new D1Dialect({ database: db }),
      type: 'sqlite'
    },

    // 이메일/비밀번호 인증
    emailAndPassword: {
      enabled: true,
      requireEmailVerification: false
    },

    // 소셜 로그인
    socialProviders: {
      google: {
        clientId: process.env.GOOGLE_CLIENT_ID!,
        clientSecret: process.env.GOOGLE_CLIENT_SECRET!
      }
    },

    // 세션 설정
    session: {
      expiresIn: 60 * 60 * 24 * 7, // 7일
      updateAge: 60 * 60 * 24      // 1일마다 갱신
    }
  });
}

export type Auth = ReturnType<typeof createAuth>;
```

### SvelteKit 통합

```typescript
// src/hooks.server.ts
import { createAuth } from '$lib/server/auth';
import type { Handle } from '@sveltejs/kit';

/**
 * 서버 훅 - 모든 요청에서 실행
 */
export const handle: Handle = async ({ event, resolve }) => {
  const db = event.platform?.env.DB;

  if (!db) {
    return resolve(event);
  }

  // Auth 인스턴스 생성
  const auth = createAuth(db);

  // 세션 검증
  const session = await auth.api.getSession({
    headers: event.request.headers
  });

  // locals에 저장 (모든 라우트에서 접근 가능)
  event.locals.auth = auth;
  event.locals.session = session;
  event.locals.user = session?.user ?? null;

  return resolve(event);
};
```

### 타입 정의
```typescript
// src/app.d.ts
import type { Auth } from '$lib/server/auth';

declare global {
  namespace App {
    interface Locals {
      auth: Auth;
      session: Session | null;
      user: User | null;
    }
  }
}
```

### API 라우트
```typescript
// src/routes/api/auth/[...all]/+server.ts
import type { RequestHandler } from './$types';

/**
 * Better Auth API 핸들러
 * /api/auth/* 경로의 모든 요청 처리
 */
export const GET: RequestHandler = async ({ request, locals }) => {
  return locals.auth.handler(request);
};

export const POST: RequestHandler = async ({ request, locals }) => {
  return locals.auth.handler(request);
};
```

### 클라이언트 사용
```typescript
// src/lib/auth-client.ts
import { createAuthClient } from 'better-auth/svelte';

/**
 * 클라이언트 사이드 Auth 인스턴스
 */
export const authClient = createAuthClient({
  baseURL: '/api/auth'
});

// 제공되는 함수들
export const {
  signIn,      // 로그인
  signUp,      // 회원가입
  signOut,     // 로그아웃
  useSession   // 세션 상태 (반응형)
} = authClient;
```

### 로그인 폼 예시
```svelte
<!-- src/routes/login/+page.svelte -->
<script lang="ts">
  import { signIn } from '$lib/auth-client';
  import { goto } from '$app/navigation';

  let email = $state('');
  let password = $state('');
  let loading = $state(false);
  let error = $state('');

  /**
   * 이메일/비밀번호 로그인
   */
  async function handleLogin() {
    loading = true;
    error = '';

    const result = await signIn.email({
      email,
      password
    });

    if (result.error) {
      error = result.error.message;
      loading = false;
      return;
    }

    goto('/dashboard');
  }

  /**
   * Google 소셜 로그인
   */
  async function handleGoogleLogin() {
    await signIn.social({
      provider: 'google',
      callbackURL: '/dashboard'
    });
  }
</script>

<form onsubmit={handleLogin}>
  <input type="email" bind:value={email} placeholder="이메일" />
  <input type="password" bind:value={password} placeholder="비밀번호" />

  {#if error}
    <p class="error">{error}</p>
  {/if}

  <button type="submit" disabled={loading}>
    {loading ? '로그인 중...' : '로그인'}
  </button>
</form>

<button onclick={handleGoogleLogin}>
  Google로 로그인
</button>
```

### 보호된 라우트
```typescript
// src/routes/(app)/+layout.server.ts
import { redirect } from '@sveltejs/kit';
import type { LayoutServerLoad } from './$types';

/**
 * 인증 필요 라우트 그룹
 * 로그인하지 않은 사용자는 /login으로 리다이렉트
 */
export const load: LayoutServerLoad = async ({ locals, url }) => {
  if (!locals.user) {
    throw redirect(303, `/login?redirectTo=${url.pathname}`);
  }

  return {
    user: locals.user
  };
};
```

---
