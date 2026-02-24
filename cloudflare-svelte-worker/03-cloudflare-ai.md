---
name: cloudflare-ai
description: Cloudflare AI Gateway를 활용한 SvelteKit AI 기능 구현 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, cloudflare, ai, workers-ai, streaming]
---

# Cloudflare AI Gateway

### 개요

AI Gateway는 AI 요청을 프록시하여 캐싱, 속도 제한, 분석 기능을 제공합니다.

### wrangler.jsonc 설정
```jsonc
{
  "ai": {
    "binding": "AI"
  },
  "vars": {
    "AI_GATEWAY_SLUG": "my-gateway"
  }
}
```

### 기본 사용법
```typescript
// src/lib/server/ai.ts

/**
 * AI Gateway를 통한 텍스트 생성
 */
export async function generateText(
  ai: Ai,
  prompt: string,
  options?: {
    model?: string;
    maxTokens?: number;
  }
) {
  const model = options?.model ?? '@cf/meta/llama-2-7b-chat-int8';

  const response = await ai.run(model, {
    messages: [
      { role: 'user', content: prompt }
    ],
    max_tokens: options?.maxTokens ?? 256
  });

  return response;
}

/**
 * 이미지 생성
 */
export async function generateImage(
  ai: Ai,
  prompt: string
) {
  const response = await ai.run(
    '@cf/stabilityai/stable-diffusion-xl-base-1.0',
    { prompt }
  );

  return response; // Uint8Array (PNG 이미지)
}

/**
 * 텍스트 임베딩 생성
 */
export async function createEmbedding(
  ai: Ai,
  text: string
) {
  const response = await ai.run(
    '@cf/baai/bge-base-en-v1.5',
    { text: [text] }
  );

  return response.data[0]; // number[]
}
```

### API 라우트 예시
```typescript
// src/routes/api/chat/+server.ts
import type { RequestHandler } from './$types';
import { json } from '@sveltejs/kit';

/**
 * 챗봇 API 엔드포인트
 */
export const POST: RequestHandler = async ({ request, platform }) => {
  const ai = platform?.env.AI;

  if (!ai) {
    return json({ error: 'AI not available' }, { status: 500 });
  }

  const { message } = await request.json();

  const response = await ai.run('@cf/meta/llama-2-7b-chat-int8', {
    messages: [
      {
        role: 'system',
        content: '당신은 친절한 한국어 AI 어시스턴트입니다.'
      },
      {
        role: 'user',
        content: message
      }
    ]
  });

  return json({ reply: response.response });
};
```

### 스트리밍 응답
```typescript
// src/routes/api/chat/stream/+server.ts
import type { RequestHandler } from './$types';

/**
 * 스트리밍 챗 응답
 */
export const POST: RequestHandler = async ({ request, platform }) => {
  const ai = platform?.env.AI;
  const { message } = await request.json();

  const stream = await ai.run('@cf/meta/llama-2-7b-chat-int8', {
    messages: [{ role: 'user', content: message }],
    stream: true
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache'
    }
  });
};
```

### 클라이언트에서 스트리밍 처리
```svelte
<script lang="ts">
  let response = $state('');
  let loading = $state(false);

  /**
   * 스트리밍 응답 받기
   */
  async function sendMessage(message: string) {
    loading = true;
    response = '';

    const res = await fetch('/api/chat/stream', {
      method: 'POST',
      body: JSON.stringify({ message })
    });

    const reader = res.body?.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader!.read();
      if (done) break;

      response += decoder.decode(value);
    }

    loading = false;
  }
</script>
```

---
