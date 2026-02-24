---
name: git-commit-guide
description: Git 커밋 메시지 작성 및 PR 생성 가이드 - Conventional Commits, Tidy First 원칙, TDD 기반 커밋 전략
category: methodology
tags: [methodology, git, commit, pr, conventional-commits, tidy-first]
---

# Git Commit & PR Guide

이 스킬은 Git 커밋 메시지 작성, PR 생성, 그리고 TDD 기반 커밋 전략을 제공합니다.

## When to Use This Skill

### 자동 트리거

- 커밋 메시지 작성 시
- PR 생성 시
- 변경 사항 커밋 전 체크리스트 확인 시
- Structural vs Behavioral 변경 분류가 필요할 시

### 수동 트리거

- `/commit` - 커밋 메시지 작성 도우미
- `/pr` - PR 생성 도우미

---

## Commit Format (Conventional Commits)

**Conventional Commits** 형식을 따릅니다.

### 기본 형식

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type (필수)

- **feat**: 새로운 기능 추가
- **fix**: 버그 수정
- **refactor**: 리팩토링 (Structural Changes)
- **test**: 테스트 추가/수정
- **docs**: 문서 수정
- **ci**: CI/CD 설정 변경
- **chore**: 기타 변경 (빌드, 의존성 등)

### Scope (선택)

모듈 또는 컴포넌트 이름 (예: `api`, `core`, `web`, `infra`)

### Subject (필수)

- 50자 이하
- 소문자로 시작
- 마침표 없음
- 명령형 현재 시제 사용

### Examples

```bash
# 새 기능
feat(api): add user profile endpoint

# 버그 수정
fix(core): correct calculation boundary order

# 리팩토링 (Structural)
refactor(api): extract calculation method
refactor(core): rename UserDto to UserResponse
refactor(api): apply SRP to UserService

# 테스트
test(core): add ScoreTest
test(api): add product detail integration test

# 문서
docs: update API guide

# CI/CD
ci: adjust pipeline cache
ci: add test coverage report
```

---

## TDD & Tidy First 커밋 전략

Kent Beck의 "Tidy First" 원칙과 TDD를 통합한 커밋 전략입니다.

### Structural Changes (구조적 변경)

**정의**: 코드의 **동작을 변경하지 않고** 구조만 개선

**Type**: `refactor`

**커밋 예시**:
```bash
refactor: extract calculation method
refactor: rename UserDto to UserResponse
refactor: move business logic to domain class
refactor: apply SRP to UserService
```

**특징**:
- 테스트 결과가 변경 전후 동일
- 행위적 변경과 절대 혼합하지 않음
- 먼저 커밋해야 함

### Behavioral Changes (행위적 변경)

**정의**: 실제 기능 추가/수정

**Type**: `feat`, `fix`, `test`

**커밋 예시**:
```bash
feat: add score calculation
fix: correct rating boundary calculation
test: add test for input validation
```

**특징**:
- TDD 사이클 (Red-Green-Refactor) 완료 후 커밋
- Structural 변경 후에 커밋
- 별도 커밋으로 분리

### 커밋 순서 규칙

```
1. Structural Changes 먼저
   └─ 커밋: "refactor: extract method for clarity"

2. Behavioral Changes 나중
   └─ 커밋: "feat: add new calculation feature"
```

**이유**: 구조를 먼저 개선하면 행위 변경이 더 쉽고 안전함

### TDD 워크플로우 예시

```
1. RED: 실패 테스트 작성
   └─ (커밋하지 않음)

2. GREEN: 최소 구현
   └─ 테스트 통과 확인
   └─ 커밋: "feat: add basic score calculation"

3. REFACTOR (Structural)
   ├─ Extract Method
   ├─ 테스트 통과 확인
   └─ 커밋: "refactor: extract calculation logic"

   ├─ Apply SOLID
   ├─ 테스트 통과 확인
   └─ 커밋: "refactor: apply SRP to calculator"

4. 다음 기능으로 1번부터 반복
```

---

## 커밋 전 체크리스트

커밋하기 전에 다음을 **반드시** 확인하세요:

### 1. 테스트 통과

- [ ] 모든 테스트가 통과하는가?
- [ ] 새로 추가한 테스트가 있는가?
- [ ] 테스트 커버리지가 유지되는가?

### 2. 컴파일/린터 경고

- [ ] 컴파일 경고가 없는가?
- [ ] 린터 경고가 없는가?

### 3. 변경 분류

- [ ] Structural 변경인가, Behavioral 변경인가?
- [ ] 두 가지가 혼합되지 않았는가?
- [ ] Structural 변경을 먼저 커밋했는가?

### 4. Conventional Commits 형식

- [ ] Type이 올바른가? (feat, fix, refactor, test, docs, ci)
- [ ] Scope가 명확한가?
- [ ] Subject가 50자 이하인가?
- [ ] 명령형 현재 시제를 사용했는가?

### 5. 커밋 크기

- [ ] 작고 빈번한 커밋인가?
- [ ] 하나의 논리적 변경만 포함하는가?
- [ ] 너무 큰 커밋이 아닌가? (100줄 이상 변경 시 분리 고려)

### 6. 커밋 메시지 품질

- [ ] 커밋 메시지가 명확한가?
- [ ] "what"과 "why"를 설명하는가?
- [ ] 미래의 나/동료가 이해할 수 있는가?

---

## Pull Request Requirements

PR을 생성하기 전에 다음을 준비하세요:

### 1. PR 제목

Conventional Commits 형식 사용:

```
feat(api): Add product detail management
fix(core): Correct score boundary calculation
refactor(api): Improve service layer structure
```

### 2. PR 본문 (템플릿)

```markdown
## Summary
[변경 사항 요약 - 1-3 문장]

## Changes
- 변경 사항 1
- 변경 사항 2
- 변경 사항 3

## Test Plan
- [ ] 단위 테스트 추가/수정
- [ ] 통합 테스트 실행
- [ ] 수동 테스트 완료
- [ ] 전체 테스트 통과

## Verification
```bash
# 실행한 명령어
```

## Related Issue
- #456

## Checklist
- [ ] 코드 리뷰 준비 완료
- [ ] 테스트 통과
- [ ] 문서 업데이트 (필요 시)
- [ ] Breaking changes 문서화 (필요 시)
```

### 3. PR 체크리스트

- [ ] **변경 사항 요약**: 무엇을 왜 변경했는가?
- [ ] **테스트 계획**: 어떻게 검증했는가?
- [ ] **검증 방법**: 실행한 명령어 기록
- [ ] **이슈 링크**: 이슈 연결
- [ ] **문서 업데이트**: 문서 변경 필요 여부 확인
- [ ] **Breaking Changes**: 하위 호환성 검토

---

## Important Rules

### 절대 금지 사항

- **NEVER** `git reset --hard` 실행
- **NEVER** `git restore` 실행 (사용자 파일 복원)
- **NEVER** 사용자가 수정한 파일을 명시적 허가 없이 되돌리기
- **NEVER** force push to main/master
- **NEVER** Structural과 Behavioral 변경을 한 커밋에 혼합

### 필수 사항

- **ALWAYS** 테스트 실행 후 커밋
- **ALWAYS** Structural 변경 먼저, Behavioral 변경 나중
- **ALWAYS** Conventional Commits 형식 준수
- **ALWAYS** 작고 빈번한 커밋
- **ALWAYS** 커밋 메시지에 의미 있는 설명

### 권장 사항

- 커밋 전 `git status` 확인
- 커밋 전 `git diff` 검토
- 의미 있는 단위로 커밋 분리
- 커밋 메시지에 이슈 번호 포함
- PR 생성 전 최신 코드 병합

---

## 고급 커밋 전략

### 커밋 분해 (Commit Decomposition)

큰 변경을 작은 커밋으로 분리:

```bash
# BAD: 모든 변경을 하나의 커밋에
git add .
git commit -m "feat: add scoring feature"

# GOOD: 논리적 단위로 분리
git add src/domain/Score.java
git commit -m "feat(core): add Score value object"

git add src/domain/Rating.java
git commit -m "feat(core): add Rating enum"

git add src/controller/ScoreController.java
git commit -m "feat(api): add score calculation endpoint"

git add src/test/ScoreControllerTest.java
git commit -m "test(api): add score controller integration test"
```

---

## Quick Reference

### Commit Types

| Type | 용도 | 예시 |
|------|------|------|
| `feat` | 새 기능 | `feat(api): add user endpoint` |
| `fix` | 버그 수정 | `fix(core): correct boundary check` |
| `refactor` | 리팩토링 | `refactor(api): extract method` |
| `test` | 테스트 | `test(core): add calculation test` |
| `docs` | 문서 | `docs: update API guide` |
| `ci` | CI/CD | `ci: add coverage report` |

### 커밋 전 체크리스트 (간단 버전)

```
- 테스트 통과
- Structural/Behavioral 분리
- Conventional Commits 형식
- 50자 이하 제목
- 작은 커밋
```

---

이 스킬을 사용하여 일관된 커밋 히스토리와 높은 품질의 PR을 유지하세요.
