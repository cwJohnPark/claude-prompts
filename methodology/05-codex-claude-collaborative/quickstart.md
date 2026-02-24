---
name: codex-claude-quickstart
description: Codex-Claude 협업 코딩 빠른 시작 가이드
category: methodology
tags: [methodology, claude, codex, quickstart]
---

# Codex-Claude Collaborative Coding - Quick Start Guide

이 가이드는 새로운 스킬을 빠르게 테스트하고 사용하는 방법을 안내합니다.

## 5분 안에 시작하기

### Step 1: Prerequisites 확인

```bash
# Codex CLI 설치 확인
codex --version
# Expected: codex-cli 0.46.0 or higher
```

### Step 2: 첫 번째 테스트 - Simple Value Object

**시나리오**: Distance Value Object 생성

```
요청: "Distance Value Object를 만들어줘. Codex와 비교해서 보여줘."
- 미터 단위 저장
- 킬로미터로 변환 가능
- Immutable
- 50줄 이하
- No else 절
```

### Step 3: 예상되는 결과

Claude Code가 다음 단계를 자동으로 실행합니다:

1. **Design Phase** - Distance VO 요구사항 분석, SOLID 원칙 체크
2. **Generation Phase** - Claude: 전통적인 Java 클래스 / Codex: Java 21 record
3. **Comparison Phase** - 비교 매트릭스 표시
4. **Presentation Phase** - Option A/B/C 제시
5. **Implementation Phase** - 사용자 선택 → 파일 작성 → 테스트
6. **Refactor Phase** - 코드 품질 검증 → 커밋

## 실전 워크플로우

### TDD와의 통합

```
1. RED: 테스트 작성 (Claude)
   → Given-When-Then 구조

2. GREEN: 구현 (Claude vs Codex 비교)
   → 병렬 생성
   → 비교 & 선택
   → 테스트 통과 확인

3. REFACTOR: 개선
   → SOLID 원칙 검증
   → 50줄 제한 확인
   → 커밋
```

## 사용 시나리오별 가이드

### Scenario 1: 새로운 도메인 모델

**When**: 새로운 Entity, Value Object, Aggregate 생성
**Approach**: Codex-Claude 비교 추천
**Reason**: 다양한 모델링 접근법 탐색

### Scenario 2: REST API 엔드포인트

**When**: 새로운 API 추가
**Approach**: Claude 우선, 복잡한 경우 비교
**Reason**: 프로젝트 패턴 엄격 준수 필요

### Scenario 3: 복잡한 알고리즘

**When**: 계산, 집계 로직 등
**Approach**: Codex-Claude 비교 강력 추천
**Reason**: 성능, 정확성, 가독성 모두 고려 필요

### Scenario 4: 테스트 케이스

**When**: 엣지 케이스 발견 필요
**Approach**: Codex 우선 (edge case 발견에 강함)

## 주의사항

### DO

- 복잡한 기능 구현 시 비교 활용
- 비교 결과를 기록
- 각 AI의 강점을 파악하며 학습
- Hybrid 접근법 적극 활용

### DON'T

- 단순 getter/setter 같은 trivial 코드에 사용하지 말 것
- Security-sensitive 코드에 사용 주의
- Production hotfix에는 검증된 방법 사용
- Codex API 비용 고려 (무분별한 사용 지양)

## Troubleshooting

### Codex 인증 실패

```bash
codex login
# 브라우저에서 ChatGPT 계정으로 로그인
```

### Codex 출력 파싱 실패

프롬프트에 출력 형식 명시:
```
"OUTPUT: Pure Java code, no markdown"
```
