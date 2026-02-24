---
name: comparison-criteria
description: Claude와 Codex 생성 코드 객관적 비교 기준표
category: methodology
tags: [methodology, claude, codex, comparison]
---

# Code Comparison Criteria

Claude와 Codex가 생성한 코드를 객관적으로 비교하기 위한 기준표입니다.

## Comparison Matrix Template

| Criteria | Claude | Codex | Winner | Notes |
|----------|--------|-------|--------|-------|
| **Line Count** | X lines | Y lines | ? | Target: <=50 lines |
| **Modern Language Features** | Traditional/Modern | Traditional/Modern | ? | records, sealed, pattern matching |
| **SOLID - SRP** | Rating | Rating | ? | Single Responsibility |
| **SOLID - OCP** | Rating | Rating | ? | Open/Closed |
| **No else Clause** | Pass/Fail | Pass/Fail | ? | Early return pattern |
| **Primitive Obsession** | Pass/Fail | Pass/Fail | ? | Value Objects used |
| **Tell, Don't Ask** | Pass/Fail | Pass/Fail | ? | Object autonomy |
| **Immutability** | Pass/Fail | Pass/Fail | ? | final fields |
| **Documentation** | Quality | Quality | ? | Doc quality |
| **Test Coverage** | X% | Y% | ? | Target: 100% |
| **Compilation** | Pass/Fail | Pass/Fail | ? | Compiles without errors |

## Scoring System

### SOLID Principles Rating

- **Excellent**: 완벽한 준수, 모범 사례
- **Good**: 준수하나 개선 여지 있음
- **Acceptable**: 최소 요구사항 충족
- **Violation**: 원칙 위반

## Category-Specific Criteria

### Value Object Comparison

| Criteria | Weight | Notes |
|----------|--------|-------|
| **Immutability** | High | final fields, unmodifiable |
| **Validation** | High | Constructor validation |
| **Static Factory** | Medium | of() pattern |
| **equals/hashCode** | High | Correct implementation |
| **No Getters** | Medium | Behavior over data |
| **Line Count <=50** | High | Mandatory |

### Entity Comparison

| Criteria | Weight | Notes |
|----------|--------|-------|
| **Identity Management** | High | ID, equals based on ID |
| **Business Methods** | High | Behavior-rich domain model |
| **No Anemic Model** | High | Avoid getter/setter only |
| **Value Objects** | Medium | Use VOs for domain concepts |
| **Aggregate Boundary** | Medium | DDD consistency boundary |

### Test Code Comparison

| Criteria | Weight | Notes |
|----------|--------|-------|
| **Given-When-Then** | High | Clear structure |
| **Coverage** | High | 100% of public methods |
| **Edge Cases** | High | Boundary, exceptions |
| **No Test Logic** | Medium | Straightforward tests |
| **Fast Execution** | Low | No external dependencies |

## Decision Making Framework

### Step 1: 필수 요구사항 검증

| Requirement | Pass/Fail |
|-------------|-----------|
| Compiles without errors | |
| <=50 lines per class | |
| No else clauses | |
| Tests exist and pass | |

**If any FAIL -> Auto-reject, request refactoring**

### Step 2: SOLID 원칙 평가

각 원칙별 점수 (0-3), Total >=12 required

### Step 3: 정성적 비교

Pattern Adherence, Innovation, Code Clarity, Test Quality

### Step 4: 최종 결정

```
1. Both pass mandatory requirements? -> Continue
2. SOLID score difference >3? -> Choose higher score
3. Significant innovation? -> Consider hybrid
4. Final recommendation: Option A/B/C
```

## Quick Decision Guide

| Scenario | Likely Winner | Reason |
|----------|---------------|--------|
| Simple Value Object | Codex | Records shine here |
| Complex Entity | Claude | DDD pattern expertise |
| API Controller | Claude | Project patterns |
| Edge Case Tests | Codex | Broader test coverage |
| Business Logic | Tie -> Hybrid | Combine strengths |
