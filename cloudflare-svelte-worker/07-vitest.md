---
name: vitest
description: Vitest를 활용한 SvelteKit 단위 테스트 및 컴포넌트 테스트 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, vitest, testing, testing-library, browser-testing]
---

# Vitest

Vite 기반의 빠른 단위 테스트 프레임워크입니다. SvelteKit과 완벽하게 통합됩니다.

### 설치

#### CLI로 프로젝트 생성 시
```bash
# Vitest 옵션 선택
pnpm dlx sv@latest create my-app
# ◼ vitest (unit testing) 선택
```

#### 기존 프로젝트에 추가
```bash
# 기본 설치 (jsdom 환경)
bun add -D vitest @testing-library/svelte @testing-library/jest-dom jsdom

# 또는 브라우저 모드 (권장 - 실제 브라우저에서 테스트)
bun add -D vitest @vitest/browser vitest-browser-svelte playwright
```

### vite.config.ts 설정

#### 기본 설정 (jsdom)
```typescript
/// <reference types="vitest/config" />
import tailwindcss from '@tailwindcss/vite';
import { sveltekit } from '@sveltejs/kit/vite';
import { svelteTesting } from '@testing-library/svelte/vite';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [tailwindcss(), sveltekit(), svelteTesting()],
  test: {
    environment: 'jsdom',
    include: ['src/**/*.{test,spec}.{js,ts}'],
    setupFiles: ['./vitest-setup.ts'],
    globals: true
  }
});
```

#### 브라우저 모드 설정 (권장)
```typescript
import tailwindcss from '@tailwindcss/vite';
import { sveltekit } from '@sveltejs/kit/vite';
import { playwright } from '@vitest/browser-playwright';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [tailwindcss(), sveltekit()],
  test: {
    projects: [
      {
        // 클라이언트 테스트 (Svelte 컴포넌트)
        extends: true,
        test: {
          name: 'client',
          testTimeout: 2000,
          browser: {
            enabled: true,
            provider: playwright(),
            instances: [{ browser: 'chromium' }]
          },
          include: ['src/**/*.svelte.{test,spec}.{js,ts}'],
          exclude: ['src/lib/server/**'],
          setupFiles: ['./vitest-setup-client.ts']
        }
      },
      {
        // 서버 테스트
        extends: true,
        test: {
          name: 'server',
          environment: 'node',
          include: ['src/**/*.server.{test,spec}.{js,ts}']
        }
      }
    ]
  }
});
```

### Setup 파일

#### vitest-setup.ts (jsdom용)
```typescript
import '@testing-library/jest-dom/vitest';
```

#### vitest-setup-client.ts (브라우저 모드용)
```typescript
import 'vitest-browser-svelte';
```

### package.json 스크립트

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  }
}
```

### 테스트 파일 명명 규칙

| 파일 패턴 | 용도 |
|-----------|------|
| `*.test.ts` | 일반 유닛 테스트 |
| `*.spec.ts` | 스펙 기반 테스트 |
| `*.svelte.test.ts` | Svelte 컴포넌트 테스트 (브라우저 모드) |
| `*.server.test.ts` | 서버 로직 테스트 |

### 기본 테스트 작성

#### 유틸리티 함수 테스트
```typescript
// src/lib/utils/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}
```

```typescript
// src/lib/utils/math.test.ts
import { describe, it, expect } from 'vitest';
import { add, multiply } from './math';

describe('math utilities', () => {
  it('should add two numbers', () => {
    expect(add(1, 2)).toBe(3);
  });

  it('should multiply two numbers', () => {
    expect(multiply(3, 4)).toBe(12);
  });
});
```

### Svelte 5 Runes 테스트

Runes를 테스트하려면 파일명에 `.svelte`가 포함되어야 합니다.

```typescript
// src/lib/stores/counter.svelte.ts
/**
 * 카운터 상태 생성 팩토리
 */
export function createCounter(initial = 0) {
  let count = $state(initial);

  return {
    get count() { return count; },
    increment() { count += 1; },
    decrement() { count -= 1; },
    reset() { count = initial; }
  };
}
```

```typescript
// src/lib/stores/counter.svelte.test.ts
import { describe, it, expect } from 'vitest';
import { flushSync } from 'svelte';
import { createCounter } from './counter.svelte';

describe('createCounter', () => {
  it('should initialize with default value', () => {
    const counter = createCounter();
    expect(counter.count).toBe(0);
  });

  it('should initialize with custom value', () => {
    const counter = createCounter(10);
    expect(counter.count).toBe(10);
  });

  it('should increment', () => {
    const counter = createCounter();

    flushSync(() => {
      counter.increment();
    });

    expect(counter.count).toBe(1);
  });

  it('should decrement', () => {
    const counter = createCounter(5);

    flushSync(() => {
      counter.decrement();
    });

    expect(counter.count).toBe(4);
  });

  it('should reset to initial value', () => {
    const counter = createCounter(10);

    flushSync(() => {
      counter.increment();
      counter.increment();
      counter.reset();
    });

    expect(counter.count).toBe(10);
  });
});
```

### Svelte 컴포넌트 테스트

#### Testing Library 방식 (jsdom)
```svelte
<!-- src/lib/components/Button.svelte -->
<script lang="ts">
  interface Props {
    label: string;
    disabled?: boolean;
    onclick?: () => void;
  }

  const { label, disabled = false, onclick }: Props = $props();
</script>

<button {disabled} {onclick}>
  {label}
</button>
```

```typescript
// src/lib/components/Button.test.ts
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/svelte';
import Button from './Button.svelte';

describe('Button', () => {
  it('should render with label', () => {
    render(Button, { props: { label: '클릭하세요' } });

    expect(screen.getByRole('button')).toHaveTextContent('클릭하세요');
  });

  it('should be disabled when disabled prop is true', () => {
    render(Button, { props: { label: '비활성', disabled: true } });

    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('should call onclick when clicked', async () => {
    const handleClick = vi.fn();
    render(Button, { props: { label: '클릭', onclick: handleClick } });

    await fireEvent.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

#### 브라우저 모드 방식 (vitest-browser-svelte)
```typescript
// src/lib/components/Button.svelte.test.ts
import { describe, it, expect, vi } from 'vitest';
import { render } from 'vitest-browser-svelte';
import { page } from '@vitest/browser/context';
import Button from './Button.svelte';

describe('Button', () => {
  it('should render with label', async () => {
    render(Button, { props: { label: '클릭하세요' } });

    const button = page.getByRole('button', { name: '클릭하세요' });
    await expect.element(button).toBeVisible();
  });

  it('should call onclick when clicked', async () => {
    const handleClick = vi.fn();
    render(Button, { props: { label: '클릭', onclick: handleClick } });

    await page.getByRole('button').click();

    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

### 비동기 테스트

```typescript
// src/lib/api/user.ts
export async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('User not found');
  return response.json();
}
```

```typescript
// src/lib/api/user.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { fetchUser } from './user';

describe('fetchUser', () => {
  beforeEach(() => {
    vi.restoreAllMocks();
  });

  it('should fetch user successfully', async () => {
    const mockUser = { id: '1', name: 'John' };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockUser)
    });

    const user = await fetchUser('1');

    expect(user).toEqual(mockUser);
    expect(fetch).toHaveBeenCalledWith('/api/users/1');
  });

  it('should throw error when user not found', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false
    });

    await expect(fetchUser('999')).rejects.toThrow('User not found');
  });
});
```

### Mock 사용법

#### 함수 모킹
```typescript
import { vi } from 'vitest';

// 함수 스파이
const spy = vi.fn();
spy('hello');
expect(spy).toHaveBeenCalledWith('hello');

// 반환값 설정
const mock = vi.fn().mockReturnValue(42);
expect(mock()).toBe(42);

// 비동기 반환값
const asyncMock = vi.fn().mockResolvedValue('data');
await expect(asyncMock()).resolves.toBe('data');

// 구현 모킹
const impl = vi.fn().mockImplementation((x) => x * 2);
expect(impl(5)).toBe(10);
```

#### 모듈 모킹
```typescript
// 전체 모듈 모킹
vi.mock('$app/navigation', () => ({
  goto: vi.fn(),
  invalidate: vi.fn()
}));

// 부분 모킹
vi.mock('./utils', async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    someFunction: vi.fn()
  };
});
```

#### 타이머 모킹
```typescript
import { vi, beforeEach, afterEach } from 'vitest';

beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

it('should handle setTimeout', async () => {
  const callback = vi.fn();

  setTimeout(callback, 1000);

  expect(callback).not.toHaveBeenCalled();

  await vi.advanceTimersByTimeAsync(1000);

  expect(callback).toHaveBeenCalled();
});
```

### SvelteKit 모듈 모킹

```typescript
// $app/stores 모킹
vi.mock('$app/stores', () => ({
  page: {
    subscribe: vi.fn((fn) => {
      fn({
        url: new URL('http://localhost'),
        params: {},
        data: {}
      });
      return () => {};
    })
  },
  navigating: { subscribe: vi.fn() }
}));

// $app/environment 모킹
vi.mock('$app/environment', () => ({
  browser: true,
  dev: true,
  building: false
}));

// $env/static/public 모킹
vi.mock('$env/static/public', () => ({
  PUBLIC_API_URL: 'http://localhost:3000'
}));
```

### 테스트 유틸리티

```typescript
// src/lib/test-utils.ts
import { render } from '@testing-library/svelte';
import type { ComponentProps, SvelteComponent } from 'svelte';

/**
 * 컴포넌트 렌더링 헬퍼
 */
export function renderComponent<T extends SvelteComponent>(
  Component: new (...args: any[]) => T,
  props: ComponentProps<T>
) {
  return render(Component, { props });
}

/**
 * 테스트용 대기 함수
 */
export function wait(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

/**
 * DOM 업데이트 대기
 */
export async function tick(): Promise<void> {
  await new Promise((resolve) => setTimeout(resolve, 0));
}
```

### 커버리지 설정

```typescript
// vite.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      include: ['src/lib/**/*.{ts,svelte}'],
      exclude: [
        'src/lib/**/*.test.ts',
        'src/lib/**/*.spec.ts',
        'src/lib/**/index.ts'
      ],
      thresholds: {
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80
      }
    }
  }
});
```

### GitHub Actions CI

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest

      - name: Install dependencies
        run: bun install

      - name: Run tests
        run: bun run test:run

      - name: Run coverage
        run: bun run test:coverage
```

### 주요 Matcher

| Matcher | 설명 |
|---------|------|
| `toBe(value)` | 엄격한 동등성 (===) |
| `toEqual(value)` | 깊은 동등성 |
| `toBeNull()` | null 체크 |
| `toBeDefined()` | undefined가 아님 |
| `toBeTruthy()` | truthy 값 |
| `toBeFalsy()` | falsy 값 |
| `toContain(item)` | 배열/문자열 포함 |
| `toHaveLength(n)` | 길이 체크 |
| `toThrow()` | 에러 발생 |
| `toHaveBeenCalled()` | 함수 호출됨 |
| `toHaveBeenCalledWith()` | 특정 인자로 호출됨 |
| `toHaveBeenCalledTimes(n)` | n번 호출됨 |

### jest-dom Matcher (DOM 테스트용)

| Matcher | 설명 |
|---------|------|
| `toBeInTheDocument()` | DOM에 존재 |
| `toBeVisible()` | 화면에 보임 |
| `toBeDisabled()` | 비활성화 상태 |
| `toHaveTextContent(text)` | 텍스트 포함 |
| `toHaveAttribute(attr, value?)` | 속성 확인 |
| `toHaveClass(className)` | 클래스 포함 |
| `toHaveFocus()` | 포커스 상태 |
| `toBeChecked()` | 체크됨 (checkbox/radio) |

### 참고 사이트

- [Vitest 공식 문서](https://vitest.dev/)
- [Testing Library Svelte](https://testing-library.com/docs/svelte-testing-library/intro)
- [Svelte Testing 가이드](https://svelte.dev/docs/svelte/testing)
- [vitest-browser-svelte](https://github.com/vitest-dev/vitest-browser-svelte)

---
