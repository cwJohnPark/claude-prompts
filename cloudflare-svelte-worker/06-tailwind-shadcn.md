---
name: tailwind-shadcn
description: Tailwind CSS v4와 shadcn-svelte UI 컴포넌트 설정 및 사용 가이드
category: cloudflare-svelte-worker
tags: [svelte, sveltekit, tailwindcss, shadcn, ui, components, dark-mode]
---

# Tailwind CSS v4 + shadcn-svelte

### 개요

**shadcn-svelte**는 복사하여 사용하는 재사용 가능한 UI 컴포넌트 라이브러리입니다.
Tailwind CSS v4와 Svelte 5를 기반으로 하며, 컴포넌트 코드가 프로젝트에 직접 추가됩니다.

### 초기 설정

#### 1. SvelteKit 프로젝트에 Tailwind CSS v4 추가
```bash
# Tailwind CSS v4 설치
bun add -D tailwindcss @tailwindcss/vite
```

#### 2. vite.config.ts 설정
```typescript
// vite.config.ts
import { sveltekit } from '@sveltejs/kit/vite';
import tailwindcss from '@tailwindcss/vite';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    tailwindcss(),  // Tailwind v4는 Vite 플러그인 사용
    sveltekit()
  ]
});
```

#### 3. shadcn-svelte 초기화
```bash
# shadcn-svelte 초기화 (대화형)
npx shadcn-svelte@latest init

# 질문에 대한 권장 답변:
# - TypeScript: Yes
# - Style: new-york (권장)
# - Base color: Slate 또는 Zinc
# - Global CSS: src/app.css
# - CSS variables: Yes
# - Import alias (components): $lib/components
# - Import alias (utils): $lib/utils
```

#### 4. app.css 구조 (Tailwind v4)
```css
/* src/app.css */
@import "tailwindcss";
@import "tw-animate-css";

/* 다크 모드 커스텀 variant */
@custom-variant dark (&:is(.dark *));

/* CSS 변수 기반 테마 (OKLCH 색상) */
:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.205 0 0);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.985 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(0.269 0 0);
  --input: oklch(0.269 0 0);
  --ring: oklch(0.439 0 0);
}
```

### 컴포넌트 추가

```bash
# 개별 컴포넌트 추가
npx shadcn-svelte@latest add button
npx shadcn-svelte@latest add card
npx shadcn-svelte@latest add input
npx shadcn-svelte@latest add dialog
npx shadcn-svelte@latest add dropdown-menu
npx shadcn-svelte@latest add form
npx shadcn-svelte@latest add table
npx shadcn-svelte@latest add tabs
npx shadcn-svelte@latest add toast

# 여러 컴포넌트 한번에 추가
npx shadcn-svelte@latest add button card input textarea label

# 모든 컴포넌트 추가
npx shadcn-svelte@latest add --all
```

### 유틸리티 함수 (cn)

```typescript
// src/lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

/**
 * Tailwind 클래스를 조건부로 결합하고 병합하는 유틸리티
 * @param inputs - 클래스 이름 또는 조건부 클래스 객체
 * @returns 병합된 클래스 문자열
 *
 * @example
 * cn('px-4 py-2', isActive && 'bg-blue-500', { 'opacity-50': disabled })
 */
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### 주요 컴포넌트 사용법

#### Button
```svelte
<script lang="ts">
  import { Button } from "$lib/components/ui/button";
</script>

<!-- 기본 버튼 -->
<Button>기본 버튼</Button>

<!-- 변형 (variants) -->
<Button variant="default">Default</Button>
<Button variant="destructive">삭제</Button>
<Button variant="outline">Outline</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="link">Link</Button>

<!-- 크기 (sizes) -->
<Button size="default">기본</Button>
<Button size="sm">작은</Button>
<Button size="lg">큰</Button>
<Button size="icon">🔔</Button>

<!-- 비활성화 -->
<Button disabled>비활성화</Button>

<!-- 로딩 상태 -->
<Button disabled>
  <Loader2 class="mr-2 size-4 animate-spin" />
  처리 중...
</Button>
```

#### Card
```svelte
<script lang="ts">
  import * as Card from "$lib/components/ui/card";
</script>

<Card.Root>
  <Card.Header>
    <Card.Title>카드 제목</Card.Title>
    <Card.Description>카드 설명 텍스트입니다.</Card.Description>
  </Card.Header>
  <Card.Content>
    <p>카드 본문 내용</p>
  </Card.Content>
  <Card.Footer>
    <Button>확인</Button>
  </Card.Footer>
</Card.Root>
```

#### Input & Label
```svelte
<script lang="ts">
  import { Input } from "$lib/components/ui/input";
  import { Label } from "$lib/components/ui/label";

  let email = $state('');
</script>

<div class="grid w-full max-w-sm gap-1.5">
  <Label for="email">이메일</Label>
  <Input
    type="email"
    id="email"
    placeholder="example@email.com"
    bind:value={email}
  />
</div>
```

#### Dialog (모달)
```svelte
<script lang="ts">
  import * as Dialog from "$lib/components/ui/dialog";
  import { Button } from "$lib/components/ui/button";
</script>

<Dialog.Root>
  <Dialog.Trigger asChild let:builder>
    <Button builders={[builder]}>모달 열기</Button>
  </Dialog.Trigger>
  <Dialog.Content>
    <Dialog.Header>
      <Dialog.Title>모달 제목</Dialog.Title>
      <Dialog.Description>
        모달 설명 텍스트입니다.
      </Dialog.Description>
    </Dialog.Header>
    <div class="py-4">
      <p>모달 본문 내용</p>
    </div>
    <Dialog.Footer>
      <Button variant="outline">취소</Button>
      <Button>확인</Button>
    </Dialog.Footer>
  </Dialog.Content>
</Dialog.Root>
```

#### Form (superforms 연동)
```svelte
<script lang="ts">
  import * as Form from "$lib/components/ui/form";
  import { Input } from "$lib/components/ui/input";
  import { superForm } from "sveltekit-superforms";
  import { zodClient } from "sveltekit-superforms/adapters";
  import { z } from "zod";

  /** 폼 스키마 정의 */
  const formSchema = z.object({
    username: z.string().min(2, "최소 2자 이상 입력하세요"),
    email: z.string().email("올바른 이메일을 입력하세요")
  });

  const { data } = $props();

  const form = superForm(data.form, {
    validators: zodClient(formSchema)
  });

  const { form: formData, enhance } = form;
</script>

<form method="POST" use:enhance>
  <Form.Field {form} name="username">
    <Form.Control let:attrs>
      <Form.Label>사용자명</Form.Label>
      <Input {...attrs} bind:value={$formData.username} />
    </Form.Control>
    <Form.FieldErrors />
  </Form.Field>

  <Form.Field {form} name="email">
    <Form.Control let:attrs>
      <Form.Label>이메일</Form.Label>
      <Input type="email" {...attrs} bind:value={$formData.email} />
    </Form.Control>
    <Form.FieldErrors />
  </Form.Field>

  <Button type="submit">제출</Button>
</form>
```

#### Dropdown Menu
```svelte
<script lang="ts">
  import * as DropdownMenu from "$lib/components/ui/dropdown-menu";
  import { Button } from "$lib/components/ui/button";
</script>

<DropdownMenu.Root>
  <DropdownMenu.Trigger asChild let:builder>
    <Button variant="outline" builders={[builder]}>메뉴 열기</Button>
  </DropdownMenu.Trigger>
  <DropdownMenu.Content class="w-56">
    <DropdownMenu.Label>내 계정</DropdownMenu.Label>
    <DropdownMenu.Separator />
    <DropdownMenu.Group>
      <DropdownMenu.Item>
        프로필
        <DropdownMenu.Shortcut>⇧⌘P</DropdownMenu.Shortcut>
      </DropdownMenu.Item>
      <DropdownMenu.Item>
        설정
        <DropdownMenu.Shortcut>⌘S</DropdownMenu.Shortcut>
      </DropdownMenu.Item>
    </DropdownMenu.Group>
    <DropdownMenu.Separator />
    <DropdownMenu.Item>
      로그아웃
      <DropdownMenu.Shortcut>⇧⌘Q</DropdownMenu.Shortcut>
    </DropdownMenu.Item>
  </DropdownMenu.Content>
</DropdownMenu.Root>
```

#### Table
```svelte
<script lang="ts">
  import * as Table from "$lib/components/ui/table";

  interface Invoice {
    id: string;
    status: string;
    method: string;
    amount: number;
  }

  const { invoices } = $props<{ invoices: Invoice[] }>();
</script>

<Table.Root>
  <Table.Header>
    <Table.Row>
      <Table.Head>Invoice</Table.Head>
      <Table.Head>Status</Table.Head>
      <Table.Head>Method</Table.Head>
      <Table.Head class="text-right">Amount</Table.Head>
    </Table.Row>
  </Table.Header>
  <Table.Body>
    {#each invoices as invoice}
      <Table.Row>
        <Table.Cell class="font-medium">{invoice.id}</Table.Cell>
        <Table.Cell>{invoice.status}</Table.Cell>
        <Table.Cell>{invoice.method}</Table.Cell>
        <Table.Cell class="text-right">{invoice.amount}</Table.Cell>
      </Table.Row>
    {/each}
  </Table.Body>
</Table.Root>
```

### 다크 모드 설정

#### mode-watcher 사용
```bash
bun add mode-watcher
```

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
  import "../app.css";
  import { ModeWatcher } from "mode-watcher";

  let { children } = $props();
</script>

<!-- 다크 모드 자동 감지 및 적용 -->
<ModeWatcher />

{@render children()}
```

#### 다크 모드 토글 버튼
```svelte
<script lang="ts">
  import { toggleMode } from "mode-watcher";
  import { Button } from "$lib/components/ui/button";
  import Sun from "lucide-svelte/icons/sun";
  import Moon from "lucide-svelte/icons/moon";
</script>

<Button onclick={toggleMode} variant="outline" size="icon">
  <Sun class="size-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
  <Moon class="absolute size-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
  <span class="sr-only">테마 전환</span>
</Button>
```

### 아이콘 (Lucide)

```bash
bun add lucide-svelte
```

```svelte
<script lang="ts">
  import Home from "lucide-svelte/icons/home";
  import Settings from "lucide-svelte/icons/settings";
  import User from "lucide-svelte/icons/user";
  import Loader2 from "lucide-svelte/icons/loader-2";
</script>

<!-- 기본 사용 -->
<Home class="size-6" />

<!-- 크기 조절 -->
<Settings class="size-4" />
<Settings class="size-8" />

<!-- 색상 -->
<User class="size-6 text-blue-500" />

<!-- 애니메이션 -->
<Loader2 class="size-6 animate-spin" />
```

### 커스텀 컴포넌트 확장

```svelte
<!-- src/lib/components/ui/button-loading.svelte -->
<script lang="ts">
  import { Button, type ButtonProps } from "$lib/components/ui/button";
  import Loader2 from "lucide-svelte/icons/loader-2";

  interface LoadingButtonProps extends ButtonProps {
    /** 로딩 상태 */
    loading?: boolean;
    /** 로딩 중 표시할 텍스트 */
    loadingText?: string;
  }

  let {
    loading = false,
    loadingText = "처리 중...",
    children,
    disabled,
    ...restProps
  }: LoadingButtonProps = $props();
</script>

<Button {...restProps} disabled={loading || disabled}>
  {#if loading}
    <Loader2 class="mr-2 size-4 animate-spin" />
    {loadingText}
  {:else}
    {@render children?.()}
  {/if}
</Button>
```

### 디렉토리 구조

```
src/lib/
├── components/
│   └── ui/              # shadcn-svelte 컴포넌트
│       ├── button/
│       │   ├── button.svelte
│       │   └── index.ts
│       ├── card/
│       │   ├── card.svelte
│       │   ├── card-header.svelte
│       │   ├── card-title.svelte
│       │   ├── card-description.svelte
│       │   ├── card-content.svelte
│       │   ├── card-footer.svelte
│       │   └── index.ts
│       ├── input/
│       ├── dialog/
│       └── ...
└── utils.ts             # cn() 유틸리티
```

### components.json 설정
```json
{
  "$schema": "https://shadcn-svelte.com/schema.json",
  "style": "new-york",
  "tailwind": {
    "config": "",
    "css": "src/app.css",
    "baseColor": "slate"
  },
  "aliases": {
    "components": "$lib/components",
    "utils": "$lib/utils",
    "ui": "$lib/components/ui",
    "hooks": "$lib/hooks"
  },
  "typescript": true
}
```

### Tailwind v4 주요 변경사항

| v3 | v4 |
|----|----|
| `tailwind.config.js` | CSS 기반 설정 (`@theme`) |
| PostCSS 플러그인 | Vite 플러그인 권장 |
| `tailwindcss-animate` | `tw-animate-css` |
| HSL 색상 | OKLCH 색상 |
| `@apply` | 여전히 지원, `@theme inline` 사용 가능 |

### 참고 사이트

- [shadcn-svelte 공식 문서](https://www.shadcn-svelte.com/)
- [shadcn-svelte 데모](https://v4.shadcn-svelte.com/)
- [Tailwind CSS v4 문서](https://tailwindcss.com/docs)
- [Lucide Icons](https://lucide.dev/icons/)

---
