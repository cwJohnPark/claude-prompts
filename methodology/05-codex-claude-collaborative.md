---
name: codex-claude-collaborative-coding
description: Claude Code와 OpenAI Codex CLI를 동시에 활용한 협업 코딩 워크플로우 - 비교 생성, 분석, 최적 솔루션 선택
category: methodology
tags: [methodology, ai-collaboration, codex, claude, code-generation, tdd]
allowed-tools: Bash, Read, Write, Grep, Glob
---

# Codex-Claude Collaborative Coding Skill

이 스킬은 Claude Code와 OpenAI Codex CLI를 동시에 사용하여 코드를 생성하고 비교하는 협업 코딩 워크플로우를 제공합니다.

## 핵심 가치

**왜 두 AI를 동시에 사용하는가?**

- **다양한 관점**: Claude와 Codex는 서로 다른 강점을 가짐
- **최선의 선택**: 여러 솔루션을 비교하여 최적의 접근법 선택
- **학습 기회**: 두 AI의 코드 스타일과 패턴을 비교하며 학습
- **품질 향상**: 경쟁적 생성으로 더 나은 코드 품질 확보

## When to Use This Skill

### Automatic Triggers

**TDD Green Stage 구현 단계**:
- 새로운 도메인 클래스 생성 (Entity, Value Object, Aggregate)
- 복잡한 비즈니스 로직 구현
- 서비스 레이어 메서드 작성
- API 컨트롤러 생성

**High Complexity Indicators**:
- 메서드 파라미터 >3개
- 클래스가 40줄 근접 (50줄 제한 고려)
- 복잡한 알고리즘 구현
- 여러 디자인 패턴 적용 가능한 경우

## Prerequisites

### Codex CLI 설치 확인

```bash
which codex
codex --version  # codex-cli 0.46.0+
```

설치되지 않은 경우:
```bash
npm install -g @openai/codex
```

## Collaborative Workflow

### Phase 1: Analysis & Design

1. **요구사항 파싱** - 핵심 기능 추출, DDD 패턴 식별
2. **설계 원칙 적용** - SOLID, 객체지향 체조 9원칙
3. **수용 기준 정의**

### Phase 2: Parallel Generation

- **Claude Path**: 프로젝트 패턴 준수, OOP/DDD 원칙, 테스트 우선
- **Codex Path**: Codex CLI로 독립 생성, 최신 언어 기능 활용

### Phase 3: Comparative Analysis

비교 매트릭스를 통해 객관적으로 두 솔루션을 비교합니다.
상세 비교 기준은 [comparison-criteria.md](./05-codex-claude-collaborative/comparison-criteria.md) 참조.

### Phase 4: Present Options

사용자에게 Option A (Claude), Option B (Codex), Option C (Hybrid) 제시

### Phase 5: Implementation

선택된 솔루션 적용, 테스트 생성 및 실행

### Phase 6: Refactor & Document

코드 품질 최종 검증 및 문서화

## Integration with Existing Skills

- **oop-ddd-refactoring**: 두 솔루션 모두 SOLID/체조 원칙으로 검증
- **git-commit-guide**: Tidy First 커밋 전략 적용

## Safety & Best Practices

### When NOT to Use

- Security-sensitive 코드 (인증, 암호화)
- 데이터베이스 마이그레이션
- Production hotfix
- 단순 getter/setter

### When to Use

- 새로운 도메인 모델
- 복잡한 알고리즘
- 대안적 설계 탐색
- 새로운 패턴 학습

## Quick Reference

### Decision Matrix

| Use Case | Approach | Reason |
|----------|----------|--------|
| Value Object | Compare | 다양한 구현 방식 탐색 |
| Entity | Compare | 대안적 도메인 모델링 |
| API Endpoint | Claude only | 프로젝트 패턴 엄격 준수 |
| Complex Algorithm | Compare | 다양한 접근법 유리 |
| Test Cases | Codex only | 엣지 케이스 발견에 강함 |
| DTO | Claude only | 문서화 + 프로젝트 패턴 |

## Related Resources

- **Quickstart**: [quickstart.md](./05-codex-claude-collaborative/quickstart.md)
- **Comparison Criteria**: [comparison-criteria.md](./05-codex-claude-collaborative/comparison-criteria.md)
- **Codex Commands**: [codex-commands-reference.md](./05-codex-claude-collaborative/codex-commands-reference.md)
