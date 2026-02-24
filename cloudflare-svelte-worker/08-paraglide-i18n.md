---
name: paraglide-i18n
description: Paraglide JS를 활용한 SvelteKit 다국어(i18n) 구현 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, paraglide, i18n, localization, multilingual]
---

# Paraglide JS (i18n)

SvelteKit 공식 i18n 통합 라이브러리입니다. 컴파일러 기반으로 tree-shakable 번역을 생성하여 최대 70% 작은 번들 크기를 제공합니다.

### 특징

| 특징 | 설명 |
|------|------|
| Tree-Shakable | 사용하지 않는 메시지는 번들러가 자동 제거 |
| 완전한 타입 안전성 | 메시지 키와 파라미터 자동완성 |
| 번들 크기 최적화 | 런타임 i18n 대비 최대 70% 감소 |
| 다양한 전략 지원 | URL, cookie, localStorage, baseLocale |
| SSR/SSG/CSR | 모든 렌더링 방식 지원 |

### 설치

```bash
npx @inlang/paraglide-js@latest init
```

CLI가 자동으로 설정합니다:
- 메시지 파일 생성 (`messages/en.json`)
- 번들러 설정 (Vite 플러그인)
- 타입 안전한 메시지 함수 생성

### vite.config.ts 설정

```typescript
import { sveltekit } from '@sveltejs/kit/vite';
import { paraglideVitePlugin } from '@inlang/paraglide-js';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    sveltekit(),
    paraglideVitePlugin({
      project: './project.inlang',
      outdir: './src/lib/paraglide',
      strategy: ['url', 'cookie', 'baseLocale']
    })
  ]
});
```

### Cloudflare Workers 설정

Edge 환경에서는 AsyncLocalStorage를 비활성화합니다:

```typescript
paraglideVitePlugin({
  project: './project.inlang',
  outdir: './src/lib/paraglide',
  strategy: ['url', 'cookie', 'baseLocale'],
  disableAsyncLocalStorage: true  // Cloudflare Workers용
})
```

### app.html 설정

```html
<!doctype html>
<html lang="%lang%">
  <head>
    <meta charset="utf-8" />
    ...
  </head>
  <body>
    %sveltekit.body%
  </body>
</html>
```

### hooks.server.ts 설정

```typescript
// src/hooks.server.ts
import type { Handle } from '@sveltejs/kit';
import { paraglideMiddleware } from '$lib/paraglide/server';

/**
 * Paraglide 미들웨어 핸들러
 * - 요청에서 로케일 감지
 * - HTML lang 속성 설정
 */
const paraglideHandle: Handle = ({ event, resolve }) =>
  paraglideMiddleware(event.request, ({ request: localizedRequest, locale }) => {
    event.request = localizedRequest;
    return resolve(event, {
      transformPageChunk: ({ html }) => html.replace('%lang%', locale)
    });
  });

export const handle: Handle = paraglideHandle;
```

### hooks.ts 설정 (reroute)

**중요**: `reroute()`는 반드시 `src/hooks.ts`에서 export해야 합니다 (hooks.server.ts 아님).

```typescript
// src/hooks.ts
import type { Reroute } from '@sveltejs/kit';
import { deLocalizeUrl } from '$lib/paraglide/runtime';

/**
 * URL에서 로케일 접두사 제거
 * /ko/about → /about
 */
export const reroute: Reroute = (request) => {
  return deLocalizeUrl(request.url).pathname;
};
```

### 메시지 파일 구조

```
messages/
├── en.json    # 영어 (기본)
├── ko.json    # 한국어
└── ja.json    # 일본어
```

#### messages/en.json
```json
{
  "greeting": "Hello {name}!",
  "welcome": "Welcome to our app",
  "items_in_cart": "{count, plural, =0 {No items} =1 {1 item} other {# items}} in cart",
  "nav": {
    "home": "Home",
    "about": "About",
    "contact": "Contact"
  }
}
```

#### messages/ko.json
```json
{
  "greeting": "안녕하세요 {name}님!",
  "welcome": "앱에 오신 것을 환영합니다",
  "items_in_cart": "장바구니에 {count}개 상품",
  "nav": {
    "home": "홈",
    "about": "소개",
    "contact": "연락처"
  }
}
```

### 기본 사용법

```svelte
<script lang="ts">
  import { m } from '$lib/paraglide/messages';
  import { getLocale, setLocale, locales } from '$lib/paraglide/runtime';

  /** 현재 로케일 */
  const currentLocale = getLocale();
</script>

<!-- 기본 메시지 -->
<h1>{m.welcome()}</h1>

<!-- 파라미터가 있는 메시지 -->
<p>{m.greeting({ name: '홍길동' })}</p>

<!-- 중첩 메시지 -->
<nav>
  <a href="/">{m.nav.home()}</a>
  <a href="/about">{m.nav.about()}</a>
</nav>

<!-- 복수형 -->
<p>{m.items_in_cart({ count: 5 })}</p>
```

### 로케일 변경

```svelte
<script lang="ts">
  import { setLocale, getLocale, locales } from '$lib/paraglide/runtime';
  import { localizeHref } from '$lib/paraglide/runtime';
  import { page } from '$app/state';

  /**
   * 로케일 변경 핸들러
   */
  function changeLocale(locale: string) {
    setLocale(locale);
  }
</script>

<!-- 방법 1: setLocale 함수 사용 -->
<select onchange={(e) => changeLocale(e.currentTarget.value)}>
  {#each locales as locale}
    <option value={locale} selected={locale === getLocale()}>
      {locale.toUpperCase()}
    </option>
  {/each}
</select>

<!-- 방법 2: 링크 기반 (SSG 권장) -->
<nav aria-label="언어 선택">
  {#each locales as locale}
    <a
      href={localizeHref(page.url.pathname, { locale })}
      data-sveltekit-reload
      aria-current={locale === getLocale() ? 'true' : undefined}
    >
      {locale.toUpperCase()}
    </a>
  {/each}
</nav>
```

### i18n 라우팅

URL 기반 로케일 라우팅이 자동으로 처리됩니다:

| URL | 로케일 |
|-----|--------|
| `/about` | 기본 로케일 (en) |
| `/ko/about` | 한국어 |
| `/ja/about` | 일본어 |

### 링크 로컬라이징

```svelte
<script lang="ts">
  import { localizeHref } from '$lib/paraglide/runtime';
</script>

<!-- 현재 로케일로 링크 -->
<a href={localizeHref('/about')}>소개</a>

<!-- 특정 로케일로 링크 -->
<a href={localizeHref('/about', { locale: 'en' })}>About (English)</a>

<!-- hreflang으로 특정 언어 강제 -->
<a href="/about" hreflang="ko">항상 한국어</a>

<!-- 번역 제외 -->
<a href="/external" data-no-translate>외부 링크</a>
```

### goto와 redirect 사용

SvelteKit의 `goto`와 `redirect`는 자동 번역되지 않습니다. `localizeHref`를 사용하세요:

```typescript
// +page.server.ts
import { redirect } from '@sveltejs/kit';
import { localizeHref } from '$lib/paraglide/runtime';

export const actions = {
  default: async ({ request }) => {
    // 로그인 처리 후 리다이렉트
    throw redirect(303, localizeHref('/dashboard'));
  }
};
```

```svelte
<script lang="ts">
  import { goto } from '$app/navigation';
  import { localizeHref } from '$lib/paraglide/runtime';

  async function navigateToProfile() {
    await goto(localizeHref('/profile'));
  }
</script>
```

### 복수형 (Pluralization)

`Intl.PluralRules` 기반으로 모든 CLDR 복수 카테고리를 지원합니다:
- zero, one, two, few, many, other

#### messages/en.json
```json
{
  "items": "{count, plural, =0 {No items} =1 {One item} other {# items}}",
  "ordinal": "{position, selectordinal, one {#st} two {#nd} few {#rd} other {#th}} place"
}
```

#### messages/ko.json
```json
{
  "items": "{count, plural, =0 {항목 없음} other {#개 항목}}",
  "ordinal": "{position}위"
}
```

```svelte
<p>{m.items({ count: 0 })}</p>  <!-- "No items" / "항목 없음" -->
<p>{m.items({ count: 1 })}</p>  <!-- "One item" / "1개 항목" -->
<p>{m.items({ count: 5 })}</p>  <!-- "5 items" / "5개 항목" -->

<p>{m.ordinal({ position: 1 })}</p>  <!-- "1st place" / "1위" -->
<p>{m.ordinal({ position: 2 })}</p>  <!-- "2nd place" / "2위" -->
```

### 정적 사이트 생성 (SSG)

```typescript
// src/routes/+layout.ts
export const prerender = true;
```

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
  import { page } from '$app/state';
  import { locales, localizeHref } from '$lib/paraglide/runtime';

  let { children } = $props();
</script>

<!-- 모든 로케일 링크 (SSG 크롤링용) -->
<nav class="locale-switcher" aria-label="Languages">
  {#each locales as locale}
    <a href={localizeHref(page.url.pathname, { locale })} data-sveltekit-reload>
      {locale}
    </a>
  {/each}
</nav>

{@render children()}
```

### project.inlang/settings.json

```json
{
  "$schema": "https://inlang.com/schema/project-settings",
  "sourceLanguageTag": "en",
  "languageTags": ["en", "ko", "ja"],
  "modules": [
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-empty-pattern@1/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-missing-translation@1/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-without-source@1/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/plugin-message-format@2/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/plugin-m-function-matcher@0/dist/index.js"
  ],
  "plugin.inlang.messageFormat": {
    "pathPattern": "./messages/{languageTag}.json"
  }
}
```

### VS Code 확장 (Sherlock)

인라인 번역 편집을 위한 VS Code 확장:

1. VS Code에서 `inlang.vs-code-extension` 설치
2. 메시지 키에 마우스 오버하면 번역 미리보기
3. 커맨드 팔레트에서 "Inlang: extract message"로 문자열 추출

### 디렉토리 구조

```
프로젝트/
├── project.inlang/
│   └── settings.json       # inlang 프로젝트 설정
├── messages/
│   ├── en.json             # 영어 메시지
│   ├── ko.json             # 한국어 메시지
│   └── ja.json             # 일본어 메시지
├── src/
│   ├── lib/
│   │   └── paraglide/      # 자동 생성 (outdir)
│   │       ├── messages.js # 컴파일된 메시지 함수
│   │       ├── runtime.js  # 로케일 관리 함수
│   │       └── server.js   # 서버 미들웨어
│   ├── hooks.ts            # reroute 훅
│   ├── hooks.server.ts     # 서버 훅 (미들웨어)
│   └── app.html            # %lang% 플레이스홀더
└── vite.config.ts          # Paraglide 플러그인 설정
```

### 런타임 API

| 함수 | 설명 |
|------|------|
| `getLocale()` | 현재 로케일 반환 |
| `setLocale(locale)` | 로케일 변경 |
| `locales` | 사용 가능한 로케일 배열 |
| `localizeHref(path, options?)` | URL 로컬라이징 |
| `deLocalizeUrl(url)` | URL에서 로케일 제거 |

### 메시지 API

| 패턴 | 설명 |
|------|------|
| `m.key()` | 기본 메시지 |
| `m.key({ param })` | 파라미터가 있는 메시지 |
| `m.nested.key()` | 중첩 메시지 |
| `m["emoji-🎉"]()` | 특수 키 이름 |

### 트러블슈팅

| 문제 | 해결 방법 |
|------|-----------|
| 로케일이 적용 안됨 | hooks.server.ts에 미들웨어 확인 |
| reroute 작동 안함 | hooks.ts (not hooks.server.ts)에서 export 확인 |
| 서버에서 로케일 없음 | 요청 컨텍스트 외부에서 메시지 호출 확인 |
| Cloudflare 오류 | `disableAsyncLocalStorage: true` 설정 |

### 참고 사이트

- [Paraglide JS 공식 문서](https://inlang.com/m/gerre34r/library-inlang-paraglideJs)
- [SvelteKit 통합 가이드](https://inlang.com/m/gerre34r/library-inlang-paraglideJs/sveltekit)
- [Svelte 공식 i18n 문서](https://svelte.dev/docs/cli/paraglide)
- [Sherlock VS Code 확장](https://inlang.com/m/r7kp499g/app-inlang-ideExtension)
- [Fink 번역 에디터](https://inlang.com/m/tdozzpar/app-inlang-finkLocalizationEditor)
