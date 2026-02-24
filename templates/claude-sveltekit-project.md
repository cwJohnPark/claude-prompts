---
name: claude-sveltekit-project
description: SvelteKit + Cloudflare Workers 프로젝트용 CLAUDE.md 스타터 템플릿
category: templates
tags: [template, sveltekit, svelte, cloudflare, workers]
---

# CLAUDE.md — {ProjectName} 개발 가이드라인

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

{ProjectName}은(는) **{프로젝트 한 줄 설명}** 애플리케이션입니다.

---

## Commands

```bash
# Development
bun run dev              # Start dev server
bun run dev -- --open    # Start dev server and open browser

# Build & Deploy
bun run build            # Build for production
bun run preview          # Build and preview with Wrangler
bun run deploy           # Build and deploy to Cloudflare

# Code Quality
bun run check            # TypeScript type checking
bun run check:watch      # Type checking in watch mode
bun run lint             # Prettier check + ESLint
bun run format           # Format with Prettier

# Testing
bun run test             # Run Vitest tests
bun run test:watch       # Run tests in watch mode

# Cloudflare
bun run cf-typegen       # Generate Cloudflare Worker types
```

---

## Architecture

SvelteKit 2 + Svelte 5 application deployed to Cloudflare Workers.

### Tech Stack
- **Framework**: SvelteKit 2 with Cloudflare adapter (`@sveltejs/adapter-cloudflare`)
- **UI Library**: Svelte 5 runes syntax (`$props()`, `$state()`, `$derived()`, `$effect()`)
- **Styling**: Tailwind CSS v4 + shadcn-svelte
- **Language**: TypeScript (strict mode)
- **Runtime**: Bun
- **Hosting**: Cloudflare Workers

### Platform Bindings
- Cloudflare platform types: `src/app.d.ts` (env, cf, ctx)
- Worker configuration: `src/worker-configuration.d.ts`
- Wrangler config: `wrangler.jsonc`

### 프로젝트 구조
```
src/
├── routes/          # SvelteKit 파일 기반 라우팅
│   ├── +layout.svelte
│   ├── +page.svelte
│   └── api/         # API 엔드포인트
├── lib/
│   ├── components/  # 재사용 가능한 컴포넌트
│   ├── server/      # 서버 전용 코드
│   └── utils/       # 유틸리티 함수
└── app.d.ts         # 타입 선언
```

---

## Coding Conventions

- 주석은 한글(Korean)로 작성
- TypeScript, Svelte 초보자에게 설명한다고 가정하고 변수, 메서드, 클래스에 주석을 항상 작성
- 평탄한 구조 (중첩 방지)
- 파일 100줄 미만 유지
- SOLID 원칙 적용

### Object Calisthenics (객체지향 생활체조)

1. 한 메서드에 오직 한 단계의 들여쓰기(indent)만 한다.
2. else 예약어를 쓰지 않는다.
3. 모든 원시값과 문자열을 포장한다.
4. 한 줄에 점을 하나만 찍는다.
5. 줄여쓰지 않는다(축약 금지).
6. 모든 엔티티를 작게 유지한다.
7. 3개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.
8. 일급 콜렉션을 쓴다.
9. 게터/세터/프로퍼티를 쓰지 않는다.
10. 하나의 ts 파일은 100줄을 넘지 않는다.

---

## 스킬 참조

| 작업 유형 | 참조 스킬 | 설명 |
|----------|----------|------|
| Svelte 5 문법 | `svelte5-runes` | Svelte 5 runes 문법 가이드 |
| 인증 | `better-auth` | Better Auth 인증 구현 |
| AI 통합 | `cloudflare-ai` | Cloudflare AI 활용 |
| 분석 | `google-analytics` | Google Analytics 연동 |
| 광고 | `google-adsense` | Google AdSense 연동 |
| UI 컴포넌트 | `tailwind-shadcn` | Tailwind + shadcn 가이드 |
| 테스트 | `vitest` | Vitest 테스트 작성법 |
| 다국어 | `paraglide-i18n` | Paraglide i18n 국제화 |

---

## 참고 자료

- [SvelteKit Documentation](https://svelte.dev/docs/kit)
- [Svelte 5 Runes](https://svelte.dev/docs/svelte/what-are-runes)
- [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [shadcn-svelte](https://www.shadcn-svelte.com/)
