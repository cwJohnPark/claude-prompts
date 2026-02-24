---
name: google-analytics
description: SvelteKit에서 Google Analytics 4 설정 및 이벤트 추적 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, google-analytics, ga4, tracking]
---

# Google Analytics

### GA4 설정

#### 1. 환경 변수
```bash
# .env 또는 wrangler.jsonc vars
PUBLIC_GA_MEASUREMENT_ID=G-XXXXXXXXXX
```

#### 2. Analytics 컴포넌트
```svelte
<!-- src/lib/components/Analytics.svelte -->
<script lang="ts">
  import { page } from '$app/stores';
  import { browser } from '$app/environment';

  /** GA Measurement ID */
  const { measurementId } = $props<{ measurementId: string }>();

  /**
   * 페이지 변경 시 페이지뷰 전송
   */
  $effect(() => {
    if (!browser) return;

    const currentPath = $page.url.pathname;

    // gtag 페이지뷰 이벤트
    window.gtag?.('config', measurementId, {
      page_path: currentPath
    });
  });
</script>

<!-- Google Analytics 스크립트 -->
<svelte:head>
  <script
    async
    src={`https://www.googletagmanager.com/gtag/js?id=${measurementId}`}
  ></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', '{measurementId}');
  </script>
</svelte:head>
```

#### 3. 루트 레이아웃에 추가
```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
  import Analytics from '$lib/components/Analytics.svelte';
  import { PUBLIC_GA_MEASUREMENT_ID } from '$env/static/public';
</script>

<Analytics measurementId={PUBLIC_GA_MEASUREMENT_ID} />

<slot />
```

### 커스텀 이벤트 추적

```typescript
// src/lib/analytics.ts

/**
 * GA4 이벤트 전송 유틸리티
 */
export function trackEvent(
  eventName: string,
  parameters?: Record<string, string | number>
) {
  if (typeof window === 'undefined') return;

  window.gtag?.('event', eventName, parameters);
}

/**
 * 버튼 클릭 추적
 */
export function trackButtonClick(buttonName: string) {
  trackEvent('button_click', {
    button_name: buttonName
  });
}

/**
 * 폼 제출 추적
 */
export function trackFormSubmit(formName: string) {
  trackEvent('form_submit', {
    form_name: formName
  });
}

/**
 * 구매 완료 추적
 */
export function trackPurchase(
  transactionId: string,
  value: number,
  currency: string = 'KRW'
) {
  trackEvent('purchase', {
    transaction_id: transactionId,
    value,
    currency
  });
}
```

### 타입 정의
```typescript
// src/app.d.ts
declare global {
  interface Window {
    gtag?: (
      command: 'config' | 'event' | 'js',
      targetId: string | Date,
      config?: Record<string, unknown>
    ) => void;
    dataLayer?: unknown[];
  }
}
```

---
