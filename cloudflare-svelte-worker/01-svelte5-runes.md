---
name: svelte5-runes
description: Svelte 5 Runes 문법과 SvelteKit 라우팅 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, runes, state, derived, effect, props]
---

# Svelte 5

### Runes 기본 문법

#### $state - 반응형 상태
```svelte
<script lang="ts">
  /**
   * 카운터 상태
   * $state로 선언하면 값 변경 시 UI가 자동으로 업데이트됨
   */
  let count = $state(0);

  /**
   * 객체 상태 (깊은 반응성 지원)
   */
  let user = $state({
    name: '홍길동',
    email: 'hong@example.com'
  });

  /** 카운트 증가 */
  function increment() {
    count += 1;
  }

  /** 사용자 이름 변경 */
  function updateName(newName: string) {
    user.name = newName; // 자동으로 반응
  }
</script>
```

#### $derived - 파생 상태
```svelte
<script lang="ts">
  let width = $state(10);
  let height = $state(20);

  /**
   * 면적 (width와 height가 변경되면 자동 재계산)
   */
  const area = $derived(width * height);

  /**
   * 복잡한 파생 로직은 $derived.by 사용
   */
  const description = $derived.by(() => {
    if (area > 100) return '큰 면적';
    if (area > 50) return '중간 면적';
    return '작은 면적';
  });
</script>
```

#### $effect - 사이드 이펙트
```svelte
<script lang="ts">
  let searchQuery = $state('');

  /**
   * 검색어 변경 시 자동 실행
   * 클린업 함수 반환 가능
   */
  $effect(() => {
    console.log(`검색어: ${searchQuery}`);

    // 디바운스된 API 호출
    const timer = setTimeout(() => {
      fetchResults(searchQuery);
    }, 300);

    // 클린업 (컴포넌트 언마운트 또는 재실행 전)
    return () => clearTimeout(timer);
  });
</script>
```

#### $props - 컴포넌트 Props
```svelte
<!-- Button.svelte -->
<script lang="ts">
  /**
   * 버튼 컴포넌트 Props
   */
  interface ButtonProps {
    /** 버튼 텍스트 */
    label: string;
    /** 버튼 타입 */
    variant?: 'primary' | 'secondary';
    /** 비활성화 여부 */
    disabled?: boolean;
    /** 클릭 핸들러 */
    onclick?: () => void;
  }

  const {
    label,
    variant = 'primary',
    disabled = false,
    onclick
  } = $props<ButtonProps>();
</script>

<button
  class={variant}
  {disabled}
  {onclick}
>
  {label}
</button>
```

#### $bindable - 양방향 바인딩
```svelte
<!-- Input.svelte -->
<script lang="ts">
  /**
   * value는 부모에서 bind: 가능
   */
  let { value = $bindable('') } = $props<{ value?: string }>();
</script>

<input bind:value />

<!-- 부모 컴포넌트 -->
<script lang="ts">
  let name = $state('');
</script>

<Input bind:value={name} />
```

### SvelteKit 라우팅

#### 페이지 데이터 로드
```typescript
// src/routes/users/+page.server.ts
import type { PageServerLoad } from './$types';

/**
 * 서버에서 페이지 데이터 로드
 * Cloudflare 바인딩 접근 가능
 */
export const load: PageServerLoad = async ({ platform }) => {
  const db = platform?.env.DB;

  const { results } = await db
    .prepare('SELECT * FROM users LIMIT 10')
    .all();

  return {
    users: results
  };
};
```

```svelte
<!-- src/routes/users/+page.svelte -->
<script lang="ts">
  import type { PageData } from './$types';

  /** 서버에서 로드된 데이터 */
  const { data } = $props<{ data: PageData }>();
</script>

<ul>
  {#each data.users as user}
    <li>{user.name}</li>
  {/each}
</ul>
```

#### Form Actions
```typescript
// src/routes/login/+page.server.ts
import type { Actions } from './$types';
import { fail, redirect } from '@sveltejs/kit';

export const actions: Actions = {
  /**
   * 로그인 폼 액션
   */
  login: async ({ request, cookies }) => {
    const formData = await request.formData();
    const email = formData.get('email') as string;
    const password = formData.get('password') as string;

    // 유효성 검사
    if (!email || !password) {
      return fail(400, {
        error: '이메일과 비밀번호를 입력하세요',
        email
      });
    }

    // 인증 로직...

    throw redirect(303, '/dashboard');
  }
};
```

---
