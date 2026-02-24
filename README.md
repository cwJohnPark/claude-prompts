---
name: skills-collection
description: LLM 코딩 에이전트를 위한 기술별 스킬 컬렉션 카탈로그
---

# Skills Collection

LLM 코딩 에이전트를 위한 기술별 스킬 컬렉션.
프로젝트 고유 내용을 제거하고, 기술 카테고리별로 재구성한 오픈소스 공유 가능한 스킬 모음입니다.

## 카테고리 개요

| 카테고리 | 스킬 수 | 설명 |
|---------|--------|------|
| [agent-team/](agent-team/) | 1 | 7-Phase 에이전트 팀 개발 워크플로우 (핵심 스킬) |
| [methodology/](methodology/) | 6 | 개발 방법론 (Git, OOP/DDD, 테스트 분석, 협업 코딩) |
| [springboot/](springboot/) | 5 | Spring Boot REST API 개발 |
| [cloudflare-svelte-worker/](cloudflare-svelte-worker/) | 8 | SvelteKit + Cloudflare Workers |
| [apple/](apple/) | 13 | iOS/watchOS/macOS SwiftUI 개발 |
| [templates/](templates/) | 3 | 프로젝트 스타터 CLAUDE.md 템플릿 |

---

## 전체 스킬 목록

### Agent-Team (핵심 워크플로우)

| 스킬 | 설명 | 태그 |
|------|------|------|
| [agent-team](agent-team/SKILL.md) | Research → Plan → Annotation → Implementation → Feedback 7-Phase 개발 사이클 | `methodology`, `agent-team`, `workflow`, `tdd` |

### Methodology (개발 방법론)

| # | 스킬 | 설명 | 태그 |
|---|------|------|------|
| 01 | [agent-team-development](methodology/01-agent-team-development.md) | LLM 에이전트 팀 기반 4-Phase 개발 사이클 | `methodology`, `agent-team`, `llm` |
| 02 | [git-commit-guide](methodology/02-git-commit-guide.md) | Git 커밋 메시지, PR, Conventional Commits 가이드 | `methodology`, `git`, `commit` |
| 03 | [oop-ddd-refactoring](methodology/03-oop-ddd-refactoring.md) | OOP/DDD 리팩토링 — SOLID, 객체지향 체조, DDD 패턴 | `methodology`, `oop`, `ddd`, `refactoring` |
| 04 | [test-performance-analyzer](methodology/04-test-performance-analyzer.md) | 테스트 실행 시간 분석 및 최적화 | `methodology`, `testing`, `performance` |
| 05 | [codex-claude-collaborative](methodology/05-codex-claude-collaborative.md) | Claude + Codex CLI 협업 코딩 워크플로우 | `methodology`, `claude`, `codex`, `ai` |
| 06 | [junie-guidelines](methodology/06-junie-guidelines.md) | JetBrains Junie AI 에이전트 가이드라인 | `methodology`, `junie`, `jetbrains` |

### Spring Boot

| # | 스킬 | 설명 | 태그 |
|---|------|------|------|
| 01 | [spring-api-developer](springboot/01-spring-api-developer.md) | REST API 개발 표준 (Controller, DTO, Swagger, 테스트) | `spring-boot`, `rest-api`, `swagger` |
| 02 | [test-writing-guide](springboot/02-test-writing-guide.md) | TDD/ATDD, Given-When-Then, 레이어별 테스트 전략 | `spring-boot`, `testing`, `tdd` |
| 03 | [cucumber-bdd-guide](springboot/03-cucumber-bdd-guide.md) | Cucumber BDD 테스트, 한국어 Gherkin 문법 | `spring-boot`, `cucumber`, `bdd` |
| 04 | [guava-guide](springboot/04-guava-guide.md) | Google Guava 라이브러리 핵심 패턴 | `spring-boot`, `java`, `guava` |
| 05 | [intellij-http-client](springboot/05-intellij-http-client.md) | IntelliJ HTTP Client 파일 작성 가이드 | `spring-boot`, `intellij`, `http-client` |

### Cloudflare SvelteKit Worker

| # | 스킬 | 설명 | 태그 |
|---|------|------|------|
| 01 | [svelte5-runes](cloudflare-svelte-worker/01-svelte5-runes.md) | Svelte 5 Runes 문법과 SvelteKit 라우팅 | `svelte`, `sveltekit`, `runes` |
| 02 | [better-auth](cloudflare-svelte-worker/02-better-auth.md) | Better Auth 인증 구현 | `svelte`, `auth`, `better-auth` |
| 03 | [cloudflare-ai](cloudflare-svelte-worker/03-cloudflare-ai.md) | Cloudflare AI Gateway 활용 | `svelte`, `cloudflare`, `ai` |
| 04 | [google-analytics](cloudflare-svelte-worker/04-google-analytics.md) | Google Analytics 4 연동 | `svelte`, `analytics`, `ga4` |
| 05 | [google-adsense](cloudflare-svelte-worker/05-google-adsense.md) | Google AdSense 광고 구현 | `svelte`, `adsense`, `ads` |
| 06 | [tailwind-shadcn](cloudflare-svelte-worker/06-tailwind-shadcn.md) | Tailwind CSS v4 + shadcn-svelte | `svelte`, `tailwind`, `shadcn` |
| 07 | [vitest](cloudflare-svelte-worker/07-vitest.md) | Vitest 단위/컴포넌트 테스트 | `svelte`, `vitest`, `testing` |
| 08 | [paraglide-i18n](cloudflare-svelte-worker/08-paraglide-i18n.md) | Paraglide JS 다국어(i18n) 구현 | `svelte`, `i18n`, `paraglide` |

### Apple (SwiftUI)

| # | 스킬 | 설명 | 태그 |
|---|------|------|------|
| 01 | [swiftui-architecture](apple/01-swiftui-architecture.md) | MVVM 아키텍처, 프로젝트 구조, 의존성 주입 | `swiftui`, `ios`, `architecture`, `mvvm` |
| 02 | [swiftui-coding-rules](apple/02-swiftui-coding-rules.md) | 코딩 규칙 및 SOLID 원칙 | `swiftui`, `ios`, `coding-rules`, `solid` |
| 03 | [swiftui-testing](apple/03-swiftui-testing.md) | Unit/Integration/UI 테스트 작성법 | `swiftui`, `ios`, `testing` |
| 04 | [swiftui-uiux-design](apple/04-swiftui-uiux-design.md) | Apple HIG 기반 UI/UX 디자인 가이드 | `swiftui`, `ios`, `uiux`, `hig` |
| 05 | [swiftui-networking](apple/05-swiftui-networking.md) | URLSession, async/await, REST API, 캐싱 | `swiftui`, `ios`, `networking` |
| 06 | [swiftui-git-workflow](apple/06-swiftui-git-workflow.md) | Git 커밋 규칙, 브랜치 전략, PR 가이드 | `swiftui`, `ios`, `git` |
| 07 | [xcode-cli](apple/07-xcode-cli.md) | Xcode CLI 도구 (xcodebuild, simctl, codesign) | `swiftui`, `ios`, `xcode`, `cli` |
| 08 | [ios-i18n-guide](apple/08-ios-i18n-guide.md) | 국제화(i18n) — String Catalogs, 복수형, RTL | `swiftui`, `ios`, `i18n` |
| 09 | [storekit-iap](apple/09-storekit-iap.md) | StoreKit 2 인앱결제 및 구독 구현 | `swiftui`, `ios`, `storekit`, `iap` |
| 10 | [freemium-model](apple/10-freemium-model.md) | Freemium 비즈니스 모델 설계 및 전환율 최적화 | `swiftui`, `ios`, `freemium`, `business` |
| 11 | [autonomous-app-testing](apple/11-autonomous-app-testing.md) | iOS Simulator 자율 앱 테스트 (E2E, 스크린샷) | `swiftui`, `ios`, `e2e`, `simulator` |
| 12 | [ios-deploy](apple/12-ios-deploy.md) | App Store Connect 배포 (Archive, 심사) | `swiftui`, `ios`, `deploy`, `app-store` |
| 13 | [ios-app-icon-generator](apple/13-ios-app-icon-generator.md) | 1024x1024 원본에서 앱 아이콘 자동 생성 | `swiftui`, `ios`, `icon`, `asset` |

### Templates (프로젝트 스타터 템플릿)

| 템플릿 | 대상 | 설명 |
|--------|------|------|
| [claude-ios-project](templates/claude-ios-project.md) | iOS/watchOS SwiftUI | Xcode 프로젝트, MVVM, 스킬 참조 테이블 포함 |
| [claude-springboot-project](templates/claude-springboot-project.md) | Spring Boot | Gradle, Kotest, 4-Phase 개발 사이클 포함 |
| [claude-sveltekit-project](templates/claude-sveltekit-project.md) | SvelteKit + Cloudflare | Svelte 5, Cloudflare Workers, Tailwind CSS 포함 |

---

## 사용법

### 개별 스킬 사용

프로젝트의 `.claude/skills/` 디렉토리에 필요한 스킬 파일을 복사합니다:

```bash
# 예: SwiftUI 프로젝트에 코딩 규칙과 테스트 스킬 추가
mkdir -p .claude/skills
cp skills/apple/02-swiftui-coding-rules.md .claude/skills/
cp skills/apple/03-swiftui-testing.md .claude/skills/
```

### 프로젝트 템플릿 사용

새 프로젝트 시작 시 해당 템플릿을 `CLAUDE.md`로 복사하고 플레이스홀더를 치환합니다:

```bash
cp skills/templates/claude-ios-project.md ./CLAUDE.md
# {ProjectName}, {AppScheme} 등을 실제 값으로 치환
```

---

## 스킬 추가 가이드

새 스킬을 추가할 때는 다음 형식을 따르세요:

```markdown
---
name: {kebab-case-name}
description: {한국어 한 줄 설명}
category: {methodology|springboot|cloudflare-svelte-worker|apple}
tags: [tag1, tag2, tag3]
---

# {한국어 스킬 제목}

{개요: 이 스킬이 무엇이고 언제 사용하는지 설명하는 1단락}

## 사용 시점

- {트리거 조건 목록}

---

## {내용 섹션들}

---

## 참고 자료

- [Link](url)
```

---

## License

MIT License
