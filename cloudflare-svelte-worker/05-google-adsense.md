---
name: google-adsense
description: SvelteKit에서 Google AdSense 광고 컴포넌트 구현 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, google-adsense, ads, monetization]
---

# Google AdSense

### AdSense 설정

#### 1. 환경 변수
```bash
PUBLIC_ADSENSE_CLIENT_ID=ca-pub-XXXXXXXXXXXXXXXX
```

#### 2. AdSense 스크립트 컴포넌트
```svelte
<!-- src/lib/components/AdSenseScript.svelte -->
<script lang="ts">
  const { clientId } = $props<{ clientId: string }>();
</script>

<svelte:head>
  <script
    async
    src={`https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=${clientId}`}
    crossorigin="anonymous"
  ></script>
</svelte:head>
```

#### 3. 광고 유닛 컴포넌트
```svelte
<!-- src/lib/components/AdUnit.svelte -->
<script lang="ts">
  import { browser } from '$app/environment';
  import { onMount } from 'svelte';

  interface AdUnitProps {
    /** 광고 슬롯 ID */
    slot: string;
    /** 광고 형식 */
    format?: 'auto' | 'fluid' | 'rectangle' | 'vertical' | 'horizontal';
    /** 전체 너비 반응형 */
    fullWidthResponsive?: boolean;
    /** 커스텀 스타일 */
    style?: string;
  }

  const {
    slot,
    format = 'auto',
    fullWidthResponsive = true,
    style = 'display:block'
  } = $props<AdUnitProps>();

  /**
   * 컴포넌트 마운트 시 광고 로드
   */
  onMount(() => {
    if (!browser) return;

    try {
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    } catch (error) {
      console.error('AdSense 로드 실패:', error);
    }
  });
</script>

<ins
  class="adsbygoogle"
  {style}
  data-ad-client="ca-pub-XXXXXXXXXXXXXXXX"
  data-ad-slot={slot}
  data-ad-format={format}
  data-full-width-responsive={fullWidthResponsive}
></ins>
```

#### 4. 광고 타입별 컴포넌트

```svelte
<!-- src/lib/components/ads/BannerAd.svelte -->
<script lang="ts">
  import AdUnit from '../AdUnit.svelte';
</script>

<!-- 상단/하단 배너 광고 (728x90) -->
<div class="ad-banner">
  <AdUnit
    slot="1234567890"
    format="horizontal"
    style="display:inline-block;width:728px;height:90px"
  />
</div>

<style>
  .ad-banner {
    text-align: center;
    margin: 1rem 0;
  }
</style>
```

```svelte
<!-- src/lib/components/ads/SidebarAd.svelte -->
<script lang="ts">
  import AdUnit from '../AdUnit.svelte';
</script>

<!-- 사이드바 광고 (300x250) -->
<div class="ad-sidebar">
  <AdUnit
    slot="0987654321"
    format="rectangle"
    style="display:inline-block;width:300px;height:250px"
  />
</div>
```

```svelte
<!-- src/lib/components/ads/InArticleAd.svelte -->
<script lang="ts">
  import AdUnit from '../AdUnit.svelte';
</script>

<!-- 본문 중간 광고 (반응형) -->
<div class="ad-in-article">
  <AdUnit
    slot="1122334455"
    format="fluid"
    style="display:block;text-align:center"
  />
</div>

<style>
  .ad-in-article {
    margin: 2rem 0;
  }
</style>
```

### 루트 레이아웃 통합

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
  import Analytics from '$lib/components/Analytics.svelte';
  import AdSenseScript from '$lib/components/AdSenseScript.svelte';
  import {
    PUBLIC_GA_MEASUREMENT_ID,
    PUBLIC_ADSENSE_CLIENT_ID
  } from '$env/static/public';
</script>

<Analytics measurementId={PUBLIC_GA_MEASUREMENT_ID} />
<AdSenseScript clientId={PUBLIC_ADSENSE_CLIENT_ID} />

<slot />
```

### 타입 정의
```typescript
// src/app.d.ts
declare global {
  interface Window {
    adsbygoogle?: { push: (config: object) => void }[];
  }
}
```

### 광고 사용 예시
```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
  import BannerAd from '$lib/components/ads/BannerAd.svelte';
  import InArticleAd from '$lib/components/ads/InArticleAd.svelte';

  const { data } = $props();
</script>

<article>
  <h1>{data.post.title}</h1>

  <!-- 상단 배너 -->
  <BannerAd />

  <div class="content">
    {@html data.post.contentPart1}

    <!-- 본문 중간 광고 -->
    <InArticleAd />

    {@html data.post.contentPart2}
  </div>

  <!-- 하단 배너 -->
  <BannerAd />
</article>
```

---

## 참고 링크

- [Svelte 5 Documentation](https://svelte.dev/docs)
- [SvelteKit Documentation](https://kit.svelte.dev/docs)
- [Better Auth Documentation](https://www.better-auth.com/)
- [Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/)
- [Google Analytics 4](https://developers.google.com/analytics/devguides/collection/ga4)
- [Google AdSense](https://support.google.com/adsense/)
- [shadcn-svelte](https://www.shadcn-svelte.com/)
- [Tailwind CSS v4](https://tailwindcss.com/docs)

---
