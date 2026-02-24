---
name: templates
description: 프로젝트 스타터 템플릿 모음
---

# 프로젝트 스타터 템플릿

새 프로젝트를 시작할 때 `CLAUDE.md`의 기초로 사용할 수 있는 템플릿 모음입니다.

## 사용법

1. 프로젝트에 맞는 템플릿을 선택합니다.
2. 프로젝트 루트에 `CLAUDE.md`로 복사합니다.
3. `{ProjectName}`, `{AppScheme}` 등 플레이스홀더를 실제 값으로 치환합니다.
4. 프로젝트에 필요한 스킬을 `skills/` 디렉토리에서 복사하여 `.claude/skills/`에 배치합니다.

## 템플릿 목록

| 템플릿 | 대상 | 설명 |
|--------|------|------|
| [claude-ios-project.md](claude-ios-project.md) | iOS/watchOS SwiftUI | Xcode 프로젝트, SwiftUI, MVVM 아키텍처 |
| [claude-springboot-project.md](claude-springboot-project.md) | Spring Boot | Gradle, Kotest, Agent-Team 개발 사이클 |
| [claude-sveltekit-project.md](claude-sveltekit-project.md) | SvelteKit + Cloudflare | Svelte 5, Cloudflare Workers, Tailwind CSS |
