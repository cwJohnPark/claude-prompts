---
name: agent-team
description: Research → Plan → Annotation → Implementation → Feedback 사이클 기반 개발 워크플로우.
  계획 승인 전 코드 작성을 금지하여 구조적 통제와 효율적 토큰 사용을 달성합니다.
---

# Agent-Team Development Workflow

깊이 읽고, 계획을 쓰고, 주석으로 다듬은 뒤, 한 번에 실행하라.

Thinking(사고)과 Typing(실행)을 분리한다. 계획 승인 전에는 코드를 쓰지 않는다. 모든 Phase는 검토 가능한 산출물을 생성하고, 사용자가 승인해야 다음 Phase로 진행한다.

## When to Use This Skill

- ROADMAP.md의 다음 미완료 항목 구현 시
- 버그 수정, 리팩토링, 새 기능 등 사소하지 않은 모든 개발 작업
- `/agent-team` 명령 또는 "루프 시작" / "다음 기능" 요청 시

---

## Core Philosophy

### Human Stays in the Driver's Seat

AI는 제안하고, 인간은 결정한다. 모든 Phase는 검토 가능한 산출물을 생성한다.

- 사용자는 Claude의 제안 중 일부만 선택, 수정, 삭제할 수 있다
- 기술 선택을 재정의할 수 있다
- 어떤 Phase에서든 방향 전환을 지시할 수 있다

### Document-Centric Collaboration

`research.md`와 `plan.md`가 협업 표면(shared state)이다.

- 대화형 지시보다 문서 기반 협업이 명확하고 구조적이다
- auto-compaction 후에도 문서는 완전한 형태로 보존된다
- 문서가 곧 소통 채널이자 계약서다

### Efficient Token Usage

잘못된 리서치 → 잘못된 계획 → 잘못된 구현. 비용이 증폭된다.

- Annotation 사이클(텍스트 편집)은 Implementation 사이클(파일 생성, 테스트 실행)보다 비용이 낮다
- 거부된 계획의 비용 ≪ 거부된 코드의 비용
- 계획 단계에서 방향을 바로잡는 것이 가장 경제적이다

---

## Development Cycle Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Development Loop                      │
│                                                         │
│  Phase 1: Research ──→ Phase 2: Plan ──→ Phase 3: Annotation
│       │                    │                   │        │
│       │                    │              (1~6회 반복)   │
│       ▼                    ▼                   ▼        │
│  Phase 4: Todo List ──→ Phase 5: Implementation         │
│                                │                        │
│                                ▼                        │
│                     Phase 6: Refactoring                │
│                                │                        │
│                                ▼                        │
│                     Phase 7: Feedback                   │
│                         │         │                     │
│                    [통과] ▼    [실패] ▼                   │
│                  Loop 완료    Phase 5 복귀               │
└─────────────────────────────────────────────────────────┘
```

| Phase | Action | Output | Gate | Forbidden |
|-------|--------|--------|------|-----------|
| 1. Research | 소스 탐색, 도메인 분석, 참조 구현 식별 | `research.md` | 사용자 검토 | 코드 작성, 파일 수정 |
| 2. Plan | 구현 계획, 인수 조건, 클래스 설계 | `plan.md` | 사용자 승인 ("approved") | 코드 작성, 파일 수정 |
| 3. Annotation | 주석 반영, 계획 수정, 대안 제시 | 갱신된 `plan.md` | 사용자 전체 승인 | 코드 작성, research/plan 외 파일 수정 |
| 4. Todo List | 승인된 계획을 구조화된 체크리스트로 변환 | 체크리스트 (plan.md 내) | 사용자 검토 | 코드 작성 |
| 5. Implementation | TDD: RED → GREEN, 프로덕션 코드 + 테스트 | 소스 코드 + 테스트 | 테스트 스위트 전체 통과 | 계획 외 파일 수정, 테스트 건너뛰기 |
| 6. Refactoring | OOP 체조 9규칙 적용, 코드 개선 | 개선된 코드 | 테스트 스위트 전체 통과 | 동작 변경, 범위 외 수정 |
| 7. Feedback | E2E 검증, 결과 보고 | 검증 보고서 | 사용자 검토 | 코드 수정 |

---

## Phase 1 — Research

ROADMAP.md에서 다음 미완료 `- [ ]` 항목을 선택하고, 구현에 필요한 정보를 수집한다.

### 활동

- 관련 소스 탐색 (Glob, Grep, Read)
- 문서, API 스펙, 외부 라이브러리 분석
- 기존 패턴과 컨벤션 식별
- 참조 구현(reference implementation) 탐색 (프로젝트 내부 + 오픈소스)
- 도메인 용어와 비즈니스 규칙 정리

### Output: `research.md`

```markdown
# Research: [기능명]

## Summary
[1~3문장 요약]

## Relevant Code Paths
- `src/domain/Order.kt` — 기존 주문 도메인
- `src/service/OrderService.kt` — 주문 서비스 계층
- ...

## Domain Analysis
[도메인 용어, 비즈니스 규칙, 제약 조건]

## Reference Implementations
[프로젝트 내 유사 구현, 오픈소스 참조]

## Open Questions
- [미확정 사항 1]
- [미확정 사항 2]
```

### Gate

- `research.md`를 사용자에게 제출한다
- 사용자 검토 후 피드백 반영

### FORBIDDEN

- 코드 작성
- 파일 생성/수정 (`research.md` 제외)

---

## Phase 2 — Plan

연구 결과와 사용자 피드백을 바탕으로 구현 계획을 작성한다.

### 활동

- 구현 접근 방식 설계
- 변경 파일 목록 작성
- 인수 조건(Given/When/Then) 정의
- 클래스/인터페이스 설계
- 테스트 전략 수립
- 리스크 식별

### Output: `plan.md`

```markdown
# Plan: [기능명]

## Approach
[접근 방식 설명]

## File Changes
| Action | File | Description |
|--------|------|-------------|
| CREATE | `src/domain/Tax.kt` | 세금 값 객체 |
| MODIFY | `src/domain/Order.kt` | addTax 메서드 추가 |
| CREATE | `test/domain/TaxBehaviorSpec.kt` | 세금 도메인 테스트 |
| ... | ... | ... |

## Acceptance Criteria
### Scenario 1: [시나리오명]
- **Given** [전제 조건]
- **When** [행위]
- **Then** [기대 결과]

### Scenario 2: ...

## Class Design
[클래스 다이어그램 또는 텍스트 설계]

## Test Strategy
[테스트 계층별 전략: 도메인, 통합, E2E]

## Risks
- [리스크 1: 설명 + 완화 방안]
- [리스크 2: ...]
```

### Gate

- `plan.md`를 사용자에게 제출한다
- 사용자의 명시적 **"approved"** 신호를 대기한다

### FORBIDDEN

- 코드 작성
- 파일 생성/수정 (`plan.md` 제외)

---

## Phase 3 — Annotation Cycle

사용자가 `plan.md`에 인라인 주석을 추가하고, Claude가 이를 반영하여 계획을 다듬는 반복 과정이다. 1~6회 반복할 수 있다.

### 주석 규약

사용자는 blockquote 형식으로 주석을 추가한다:

| 주석 | 의미 | Claude의 행동 |
|------|------|---------------|
| `> Q: [질문]` | 질문 | 인라인 답변 추가 |
| `> CHANGE: [요청]` | 변경 요청 | 계획 해당 부분 수정 |
| `> CONCERN: [우려]` | 우려 사항 | 대안 제시 |
| `> REJECT: [사유]` | 접근 거부 | 대체안 작성 |
| `> APPROVED` | 해당 섹션 승인 | 섹션 확정 |

### 예시

```markdown
## Approach
Tax 계산을 Order 도메인 내부에 위임한다.

> Q: Tax 정책이 국가별로 다를 수 있는데, 확장 가능한 구조인가?
→ A: 현재 요구사항은 단일 세율이므로 Value Object로 충분합니다.
     다중 세율 필요 시 TaxPolicy 인터페이스로 확장 가능합니다.

> CHANGE: Tax를 별도 Value Object가 아니라 Money에 메서드로 추가해주세요.
```

### 핵심 규칙

Claude는 **"주석을 반영해 문서를 갱신하되, 아직 구현하지 말 것"** 규칙을 준수한다.

### Gate

- 사용자가 전체 계획에 대해 승인을 선언한다

### FORBIDDEN

- 코드 작성
- `research.md` / `plan.md` 외 파일 수정

---

## Phase 4 — Todo List

승인된 `plan.md`를 구조화된 체크리스트로 변환한다.

### 형식

TDD 순서를 반영한다 (테스트 먼저, 구현 후). 각 항목에 대상 파일과 검증 기준을 명시한다.

```markdown
## Todo List

- [ ] T001 도메인 테스트: `MoneyBehaviorSpec` — addTax 시나리오
      → File: `test/domain/MoneyBehaviorSpec.kt`
      → Verify: RED 확인 (컴파일 실패 또는 테스트 실패)
- [ ] T002 구현: `Money.addTax()` in `domain/Money.kt`
      → Verify: GREEN 확인 (T001 통과)
- [ ] T003 통합 테스트: `PriceServiceIntegrationSpec`
      → File: `test/service/PriceServiceIntegrationSpec.kt`
      → Verify: RED 확인
- [ ] T004 구현: `PriceService.calculateWithTax()`
      → Verify: GREEN 확인 (T003 통과)
- [ ] T005 컨트롤러 테스트: `PriceControllerSpec`
      → File: `test/controller/PriceControllerSpec.kt`
- [ ] T006 구현: `PriceController.getPrice()` 엔드포인트
      → Verify: 전체 테스트 스위트 통과
```

### Gate

- 사용자 검토

### FORBIDDEN

- 코드 작성

---

## Phase 5 — Implementation (TDD)

Todo 항목을 순서대로 실행한다. RED → 실패 확인 → GREEN → 통과 확인.

### 테스트 규칙

프로젝트 CLAUDE.md에 정의된 테스트 프레임워크/규칙을 따른다. Skill 내에 기술 스택별 규칙을 포함하지 않고, 프로젝트별 CLAUDE.md에서 정의하도록 안내한다.

일반적 원칙:

- BDD 프레임워크 사용 (Given / When / Then)
- Mock 최소화 — 실제 객체, 인메모리 DB, 수동 Fake 우선
- 3계층 테스트 구조:
  - 계층 1 — 순수 도메인: 프레임워크 없음, 직접 객체 생성
  - 계층 2 — 통합: 프레임워크 컨텍스트 + 인메모리 DB
  - 계층 3 — 외부 API: 수동 Fake 구현체

### 실행 프롬프트

- "모든 태스크를 완료할 때까지 멈추지 말 것"
- "불필요한 주석 금지"
- "`any`/`unknown` 타입 금지"
- "지속적 타입체크"

### 실행 중 피드백

사용자가 감독자 역할을 한다. 짧고 명확한 지시로 개입한다:

- "이 함수 누락"
- "wider"
- "2px gap"
- "이 방향 아님 — revert"

### 잘못된 방향 시

점진적 수정보다 `git revert` 후 범위를 축소하여 재시도하는 것이 효과적이다. 잘못된 코드 위에 패치를 쌓지 않는다.

### Gate

- 프로젝트 테스트 스위트 전체 통과

### FORBIDDEN

- 계획 외 파일 수정
- 테스트 건너뛰기

---

## Phase 6 — Refactoring

Implementation에서 변경한 파일에 대해서만 리팩토링을 적용한다.

### 9가지 OOP 체조 규칙

위반 발견 시 즉시 수정한다.

| # | 규칙 | 수정 방향 |
|---|------|----------|
| 1 | 반복문(`for`/`forEach`) 금지 | 함수형 API 사용 (`map`, `filter`, `firstOrNull`, `flatMap`, `fold`) |
| 2 | `else`/`else if` 금지 | early return 또는 패턴 매칭 사용 |
| 3 | 원시값으로 도메인 표현 금지 | 값 객체(Value Object)로 포장 |
| 4 | 한 줄에 점(`.`) 2개 이상 금지 | 줄바꿈으로 분리 |
| 5 | 축약 이름 금지 | 완전한 이름 사용 |
| 6 | 파일 100줄 초과 금지 | 역할별로 분리 |
| 7 | 생성자 파라미터 4개 이상 금지 | 값 객체로 병합 |
| 8 | 컬렉션 직접 노출 금지 | 일급 컬렉션으로 감싸기 |
| 9 | 외부에서 프로퍼티 꺼내 판단 금지 | 객체 내부 메서드로 이동 (Tell, Don't Ask) |

각 리팩토링 후 테스트 스위트를 실행하여 통과를 확인한다. 상세 규칙은 `oop-ddd-refactoring` 스킬을 참조한다.

### Gate

- 테스트 스위트 전체 통과

### FORBIDDEN

- 동작 변경
- 범위 외 파일 수정 (Implementation에서 변경한 파일만 대상)

---

## Phase 7 — Feedback / Verification

### 전제

앱이 `localhost:8080`에서 실행 중이어야 한다.

### 활동

Playwright MCP를 활용하여 E2E 검증을 수행한다:

1. `navigate` → 대상 페이지 이동
2. `screenshot` → 현재 상태 캡처
3. 필요 시 `click`, `fill`로 상호작용
4. 검증 결과를 보고서로 정리

### 문제 발견 시

Phase 5로 복귀하여 수정한다.

### Gate

- E2E 검증 완료 후 사용자에게 결과 보고

### FORBIDDEN

- 코드 수정 (문제 발견 시 Phase 5 복귀)

---

## Loop Completion

모든 Phase를 통과하면:

1. **커밋 제안** — Conventional Commits 형식 (`git-commit-guide` 스킬 참조)
2. **ROADMAP 갱신** — 완료 항목을 `- [x]`로 변경
3. **계속** — 사용자가 "계속"이라고 하면 Phase 1로 복귀

---

## Coding Principles

### 1. Think Before You Code

**가정하지 말 것. 혼란을 숨기지 말 것. 트레이드오프를 드러낼 것.**

구현 전에 반드시:

- 가정을 명시적으로 진술한다. 불확실하면 질문한다.
- 해석이 여러 가지라면 모두 제시한다 — 조용히 하나만 고르지 않는다.
- 더 단순한 접근이 있으면 말한다. 타당할 때는 반론을 제기한다.
- 무언가 불명확하면 멈춘다. 무엇이 혼란스러운지 명명하고 질문한다.

### 2. Simplicity First

**문제를 해결하는 최소한의 코드만 작성한다. 추측성 코드는 없다.**

- 요청받지 않은 기능을 추가하지 않는다.
- 한 번만 쓰이는 코드에 추상화를 만들지 않는다.
- 요청되지 않은 "유연성"이나 "설정 가능성"을 넣지 않는다.
- 불가능한 시나리오에 대한 에러 처리를 하지 않는다.
- 200줄로 쓴 것이 50줄로 가능하다면, 다시 쓴다.

**자문:** "시니어 엔지니어가 이걸 보고 '과도하게 복잡하다'고 할까?" 그렇다면 단순화한다.

### 3. Surgical Changes

**반드시 필요한 부분만 수정한다. 자기가 만든 잔해만 치운다.**

기존 코드를 편집할 때:

- 인접 코드, 주석, 포매팅을 "개선"하지 않는다.
- 고장 나지 않은 것을 리팩토링하지 않는다.
- 기존 스타일에 맞춘다, 자신이라면 다르게 했더라도.
- 관련 없는 죽은 코드를 발견하면 언급만 한다 — 삭제하지 않는다.

자기 변경으로 인해 고아가 된 경우:

- 자기 변경 때문에 쓸모없어진 import/변수/함수는 제거한다.
- 기존부터 있던 죽은 코드는 요청 없이 제거하지 않는다.

**검증 기준:** 변경된 모든 줄이 사용자의 요청에 직접 연결되어야 한다.

### 4. Goal-Based Execution

**성공 기준을 정의하고, 검증될 때까지 반복한다.**

작업을 검증 가능한 목표로 변환한다:

- "유효성 검사 추가" → "잘못된 입력에 대한 테스트를 작성하고, 통과시킨다"
- "버그 수정" → "재현하는 테스트를 작성하고, 통과시킨다"
- "X 리팩토링" → "리팩토링 전후로 테스트가 통과하는지 확인한다"

다단계 작업에는 간결한 계획을 명시한다:

```
1. [단계] → 검증: [확인 사항]
2. [단계] → 검증: [확인 사항]
3. [단계] → 검증: [확인 사항]
```

강한 성공 기준은 독립적으로 반복할 수 있게 해준다. 약한 기준("작동하게 해줘")은 끊임없는 명확화를 요구한다.

---

## Refactoring Rules (Quick Reference)

| # | 규칙 | 수정 방향 |
|---|------|----------|
| 1 | 반복문(`for`/`forEach`) 금지 | 함수형 API (`map`, `filter`, `firstOrNull`, `flatMap`, `fold`) |
| 2 | `else`/`else if` 금지 | early return / 패턴 매칭 |
| 3 | 원시값 도메인 표현 금지 | Value Object로 포장 |
| 4 | 한 줄에 `.` 2개 이상 금지 | 줄바꿈 분리 |
| 5 | 축약 이름 금지 | 완전한 이름 |
| 6 | 파일 100줄 초과 금지 | 역할별 분리 |
| 7 | 생성자 파라미터 4개 이상 금지 | Value Object로 병합 |
| 8 | 컬렉션 직접 노출 금지 | 일급 컬렉션 |
| 9 | 프로퍼티 꺼내 판단 금지 | Tell, Don't Ask |

프로젝트별 상세 규칙은 CLAUDE.md 또는 `oop-ddd-refactoring` 스킬을 참조한다.

---

## Test Strategy Guidelines

범용적 테스트 원칙만 포함한다. 구체적 프레임워크/라이브러리는 프로젝트 CLAUDE.md에서 정의한다.

### 3계층 테스트 구조

| 계층 | 범위 | 특징 |
|------|------|------|
| 1. 순수 도메인 | 비즈니스 로직 | 프레임워크 없음, 직접 객체 생성, 가장 빠름 |
| 2. 통합 | 서비스 + 인프라 | 프레임워크 컨텍스트, 인메모리 DB |
| 3. 외부 API | 외부 시스템 연동 | 수동 Fake 구현체 |

### BDD 스타일

- Given (전제 조건) / When (행위) / Then (기대 결과) 구조
- 시나리오명은 비즈니스 언어로 작성

### Mock 최소화

- Mock 대신 실제 객체, 인메모리 DB, 수동 Fake를 우선한다
- Mock은 외부 시스템 경계에서만 허용

---

## Integration with Other Skills

| 연계 시점 | 스킬 | 용도 |
|-----------|------|------|
| Phase 5 — Implementation | `spring-api-developer` | REST API 개발 표준 적용 |
| Phase 6 — Refactoring | `oop-ddd-refactoring` | SOLID, OOP 체조, DDD 패턴 상세 규칙 |
| Loop Completion | `git-commit-guide` | Conventional Commits, TDD 커밋 전략 |

---

## Validation Checklist

### Phase Gate 검증

- [ ] Phase 1: `research.md` 작성 완료, 사용자 검토 완료
- [ ] Phase 2: `plan.md` 작성 완료, 사용자 승인 완료
- [ ] Phase 3: Annotation 사이클 완료, 전체 계획 승인
- [ ] Phase 4: Todo List 작성 완료, 사용자 검토 완료
- [ ] Phase 5: 모든 Todo 항목 완료, 테스트 스위트 전체 통과
- [ ] Phase 6: OOP 체조 9규칙 적용 완료, 테스트 스위트 전체 통과
- [ ] Phase 7: E2E 검증 완료, 사용자에게 결과 보고

### Code Quality 검증

- [ ] OOP 체조 규칙 1: 반복문 제거
- [ ] OOP 체조 규칙 2: else 제거
- [ ] OOP 체조 규칙 3: 원시값 포장
- [ ] OOP 체조 규칙 4: 점 2개 이상 제거
- [ ] OOP 체조 규칙 5: 축약 이름 제거
- [ ] OOP 체조 규칙 6: 파일 100줄 이하
- [ ] OOP 체조 규칙 7: 생성자 파라미터 3개 이하
- [ ] OOP 체조 규칙 8: 컬렉션 일급 컬렉션으로 포장
- [ ] OOP 체조 규칙 9: Tell, Don't Ask 준수

### Process Discipline

- [ ] 계획 승인 전 코드 작성 없음
- [ ] 범위 외 파일 수정 없음
- [ ] RED → GREEN 순서 확인
- [ ] 리팩토링 후 테스트 통과 확인
- [ ] Conventional Commits 형식 준수

---

## Quick Reference

| Phase | 한 줄 요약 | Gate | 필수 문서 | Forbidden |
|-------|-----------|------|----------|-----------|
| 1. Research | 소스 탐색, 도메인 분석 | 사용자 검토 | `research.md` | 코드 작성 |
| 2. Plan | 구현 계획 작성 | 사용자 "approved" | `plan.md` | 코드 작성 |
| 3. Annotation | 주석 반영, 계획 다듬기 | 전체 승인 | 갱신된 `plan.md` | 코드 작성 |
| 4. Todo List | 체크리스트 변환 | 사용자 검토 | 체크리스트 | 코드 작성 |
| 5. Implementation | TDD: RED → GREEN | 테스트 전체 통과 | 소스 + 테스트 | 계획 외 수정 |
| 6. Refactoring | OOP 체조 9규칙 | 테스트 전체 통과 | — | 동작 변경 |
| 7. Feedback | E2E 검증 | 사용자 확인 | 검증 보고서 | 코드 수정 |

### 주요 명령어

- `/agent-team` — 이 워크플로우 시작
- "루프 시작" / "다음 기능" — Phase 1부터 시작
- "계속" — 다음 루프 시작
- `> Q:` / `> CHANGE:` / `> CONCERN:` / `> REJECT:` / `> APPROVED` — Annotation 주석

### 핵심 원칙

- **속도보다 신중함** — 기본 방침
- **계획 승인 전 코드 작성 금지** — 최우선 규칙
- **거부된 계획 비용 ≪ 거부된 코드 비용** — 경제적 근거
