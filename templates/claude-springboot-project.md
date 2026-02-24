---
name: claude-springboot-project
description: Spring Boot 프로젝트용 CLAUDE.md 스타터 템플릿
category: templates
tags: [template, spring-boot, java, kotlin, gradle]
---

# CLAUDE.md — {ProjectName} 개발 가이드라인

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**기본 방침:** 속도보다 신중함을 우선한다. 단순한 작업에는 재량껏 판단한다.

## Project Overview

{ProjectName}은(는) **{프로젝트 한 줄 설명}** 시스템입니다.

---

## Build & Run Commands

```bash
# Build
./gradlew build

# Run tests
./gradlew test

# Run application
./gradlew bootRun

# Clean
./gradlew clean
```

---

## Architecture

### Tech Stack
- **Language**: Kotlin / Java
- **Framework**: Spring Boot {버전}
- **Build Tool**: Gradle
- **Database**: {DB 종류}
- **Test**: Kotest BehaviorSpec

### 프로젝트 구조
```
src/main/kotlin/{package}/
├── domain/          # 도메인 모델, Value Object
├── service/         # 비즈니스 로직
├── controller/      # REST API 컨트롤러
├── repository/      # 데이터 접근 계층
└── config/          # 설정 클래스

src/test/kotlin/{package}/
├── domain/          # 도메인 단위 테스트
├── service/         # 통합 테스트
├── controller/      # 컨트롤러 테스트
└── support/         # Fake, Fixture, TestConfig
```

---

## 4-Phase 개발 사이클

Agent-Team 방식의 개발 사이클을 따릅니다.

### Phase 1 — 설계 (Design)
- 소스 탐색, 구현 계획 작성 (변경 파일 목록, 인수 조건 Given/When/Then)
- **게이트:** 사용자 승인 없이 Phase 2 진행 금지

### Phase 2 — 개발 (Development)
- BDD 방식의 TDD를 따른다
- **테스트 프레임워크:** Kotest `BehaviorSpec` (`given` / `when` / `then`)
- **목 라이브러리 금지:** MockK, Mockito 사용 금지 — 실제 객체, H2, 수동 Fake만 허용
- **절차:** 스펙 작성(RED) → 실패 확인 → 구현(GREEN) → 통과 확인
- **게이트:** `./gradlew test` 전체 통과 필수

### Phase 3 — 리팩토링 (Refactoring)
- 변경한 파일에 대해 객체지향 체조 규칙 체크리스트 적용
- 각 리팩토링 후 `./gradlew test` 통과 확인
- **게이트:** `./gradlew test` 전체 통과 필수

### Phase 4 — 검증 (Verification)
- E2E 검증 수행
- 문제 발견 시 Phase 2로 복귀
- **게이트:** 검증 완료 후 사용자에게 결과 보고

---

## 테스트 인프라 규칙

- **BDD 프레임워크:** Kotest `BehaviorSpec`
- **금지:** MockK, Mockito, WireMock, MockWebServer
- **허용:** 수동 Fake, `@TestConfiguration`, `@Primary`
- **테스트 3계층:**
  - 계층 1 — 순수 도메인: Spring 없음, 직접 객체 생성
  - 계층 2 — 통합: `@SpringBootTest` + H2 + `MockMvcTester`
  - 계층 3 — 외부 API: 수동 Fake 구현체 (`support/` 패키지)

---

## 스킬 참조

| 작업 유형 | 참조 스킬 | 설명 |
|----------|----------|------|
| API 개발 | `spring-api-developer` | REST API 개발 가이드 |
| 테스트 작성 | `test-writing-guide` | Kotest BDD 테스트 작성법 |
| BDD 시나리오 | `cucumber-bdd-guide` | Cucumber BDD 가이드 |
| 유틸리티 | `guava-guide` | Google Guava 활용 |
| API 테스트 | `intellij-http-client` | IntelliJ HTTP Client |
| 리팩토링 | `oop-ddd-refactoring` | 객체지향 체조 + DDD |
| 커밋 | `git-commit-guide` | Git 커밋 가이드 |

---

## 핵심 원칙

### 코딩 전 원칙 — 먼저 생각하라
- 가정을 명시적으로 진술한다. 불확실하면 질문한다.
- 해석이 여러 가지라면 모두 제시한다.
- 더 단순한 접근이 있으면 말한다.

### 단순함 우선
- 문제를 해결하는 최소한의 코드만 작성한다.
- 요청받지 않은 기능을 추가하지 않는다.
- 한 번만 쓰이는 코드에 추상화를 만들지 않는다.

### 외과적 변경
- 반드시 필요한 부분만 수정한다.
- 인접 코드, 주석, 포매팅을 "개선"하지 않는다.
- 기존 스타일에 맞춘다.
