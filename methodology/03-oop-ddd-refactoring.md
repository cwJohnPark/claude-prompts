---
name: oop-ddd-refactoring
description: OOP/DDD 기반 코드 리팩토링 스킬 - SOLID 원칙, 객체지향 체조 9원칙, DDD 패턴 검증 및 코드 스멜 제거
category: methodology
tags: [methodology, oop, ddd, solid, refactoring, tdd, object-calisthenics]
---

# OOP/DDD 리팩토링 Skill

이 스킬은 코드를 Test-Driven Development(TDD) 방법론과 객체지향 프로그래밍(OOP), 도메인 주도 설계(DDD) 원칙에 따라 리팩토링합니다.

## When to Use This Skill

### 사용 시점

이 스킬은 다음 상황에서 사용할 수 있습니다:

#### 코드 품질 검토가 필요할 때

- 복잡한 Domain Layer 코드 작성 후 리팩토링 시
- 코드 리뷰 요청 시
- 레거시 코드 개선 시
- DDD 패턴 적용 검토 시
- 기존 코드의 품질 검증 시

#### 선택적 사용

- **Design 단계**: 복잡한 기능 설계 시 참고 가능
- **Refactor 단계**: 코드 개선이 필요할 때 검증 가능

---

## TDD 방법론 참고

### 간소화된 개발 사이클

다음과 같은 실용적인 개발 사이클을 사용합니다:

1. **요구사항 확인** - 구현할 기능 명확화
2. **테스트 스케치** - 테스트 케이스 먼저 작성 (컴파일 오류 허용)
3. **코드 구현** - 테스트를 통과시키는 코드 작성
4. **테스트 완성 & 리팩토링** - 테스트 완성 및 코드 정리

### 리팩토링 시점

이 스킬은 4단계 **"테스트 완성 & 리팩토링"** 단계에서 사용할 수 있습니다:

```java
// 리팩토링 예시: SOLID 원칙과 객체지향 체조 적용
public class Order {
	private final OrderItems items;         // 일급 컬렉션
	private final Money budget;             // Value Object

	public TotalPrice calculateTotalPrice(Discount discount) {
		Money subtotal = items.calculateSubtotal();  // Tell, Don't Ask
		return TotalPrice.calculate(subtotal, budget, discount);
	}
}
```

**적용할 원칙들**:

- SOLID 원칙 (아래 섹션 참조)
- 객체지향 체조 9원칙
- DDD 패턴
- 코드 스멜 제거

**중요**: 리팩토링 후 모든 테스트가 여전히 통과해야 함

### Tidy First 접근법

Kent Beck의 "Tidy First"는 변경을 두 가지로 명확히 구분합니다:

#### 1. Structural Changes (구조적 변경)

코드의 **동작을 변경하지 않고** 구조만 개선:

- 메서드/클래스 이름 변경
- Extract Method (메서드 추출)
- Extract Class (클래스 분리)
- Move Method (메서드 이동)
- 들여쓰기 개선

**원칙**:

- 구조적 변경 전후에 테스트 실행
- 테스트 결과가 동일해야 함 (동작 불변)
- 행위적 변경과 절대 혼합하지 않음

#### 2. Behavioral Changes (행위적 변경)

실제 기능을 추가하거나 수정:

- 새로운 기능 추가
- 버그 수정
- 알고리즘 변경
- 비즈니스 로직 수정

**원칙**:

- 항상 구조적 변경을 먼저 수행
- 행위적 변경은 별도 커밋
- TDD 사이클(Red-Green-Refactor)을 따름

#### Tidy First 워크플로우

```
새 기능 요구사항
    ↓
기존 코드 구조 개선 필요?
    ↓ Yes
Structural Change (Tidy First)
    - Extract Method
    - Rename for clarity
    - 테스트 실행 → 통과 확인
    - 커밋: "refactor: extract calculation logic"
    ↓
Behavioral Change (TDD)
    - Red: 실패 테스트 작성
    - Green: 최소 구현
    - Refactor: SOLID 원칙 적용
    - 테스트 실행 → 통과 확인
    - 커밋: "feat: add score calculation"
```

---

## Refactor 단계에서 적용할 원칙들

TDD의 Refactor 단계에서는 다음 원칙들을 체계적으로 적용합니다.

---

## SOLID 원칙 검증

상세 내용은 [solid-principles-checklist.md](./03-oop-ddd-refactoring/solid-principles-checklist.md) 참조.

### 1. Single Responsibility Principle (SRP)

**정의**: 클래스는 단 하나의 책임만 가져야 하며, 변경 이유도 하나여야 한다.

**체크리스트**:

- [ ] 클래스가 50줄 이하인가?
- [ ] 클래스명이 명확한 단일 역할을 나타내는가?
- [ ] 메서드들이 모두 클래스의 주 책임과 관련이 있는가?
- [ ] 클래스 변경 시 한 가지 이유만 존재하는가?

### 2. Open/Closed Principle (OCP)

**정의**: 확장에는 열려있고 수정에는 닫혀있어야 한다.

**체크리스트**:

- [ ] 새로운 기능 추가 시 기존 코드 수정이 불필요한가?
- [ ] 인터페이스나 추상 클래스를 활용하는가?
- [ ] 전략 패턴이나 템플릿 메서드 패턴을 적용했는가?

### 3. Liskov Substitution Principle (LSP)

**정의**: 하위 타입은 상위 타입을 완전히 대체할 수 있어야 한다.

### 4. Interface Segregation Principle (ISP)

**정의**: 클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다.

### 5. Dependency Inversion Principle (DIP)

**정의**: 구체 클래스가 아닌 추상화에 의존해야 한다.

---

## 객체지향 체조 9원칙

상세 내용은 [object-calisthenics-rules.md](./03-oop-ddd-refactoring/object-calisthenics-rules.md) 참조.

### 원칙 1: 한 메서드에 오직 한 단계의 들여쓰기

깊은 들여쓰기는 가독성을 하락시키고 메서드가 너무 많은 일을 한다는 신호입니다.
**해결**: Extract Method

### 원칙 2: else 예약어 금지

**해결**: Early Return, Guard Clause, 전략 패턴

### 원칙 3: 모든 원시값과 문자열 포장

**해결**: Value Object 패턴

### 원칙 4: 한 줄에 점 하나 (디미터 법칙)

**해결**: Tell, Don't Ask

### 원칙 5: 축약 금지

### 원칙 6: 엔티티를 작게 유지

- 클래스: **50줄 이하**
- 패키지: **10개 파일 이하**

### 원칙 7: 인스턴스 변수 2개 이하

### 원칙 8: 일급 컬렉션 사용

### 원칙 9: Getter/Setter/Property 지양

**해결**: Tell, Don't Ask 원칙

---

## DDD 패턴 개선

### Entity 검증

- [ ] 고유 식별자(ID)를 가지는가?
- [ ] 생명주기가 있는가?
- [ ] 상태 변경 메서드가 비즈니스 규칙을 검증하는가?
- [ ] 불변 필드는 `final`로 선언되었는가?

### Value Object 검증

- [ ] 불변 객체인가? (모든 필드가 `final`)
- [ ] 등가성을 값으로 비교하는가? (`equals`, `hashCode`)
- [ ] 유효성 검증을 생성자에서 수행하는가?
- [ ] 원시값 대신 사용되는가? (원칙 3)

### Aggregate 경계 설정

- [ ] Aggregate Root가 명확한가?
- [ ] 외부에서 Aggregate 내부 Entity에 직접 접근하지 않는가?
- [ ] 트랜잭션 경계가 Aggregate와 일치하는가?
- [ ] Aggregate 간 참조는 ID로 하는가?

### Repository 패턴

- [ ] Aggregate Root 단위로 Repository가 존재하는가?
- [ ] 인터페이스는 도메인 계층에, 구현은 인프라 계층에 있는가?
- [ ] 컬렉션처럼 사용할 수 있는가? (`add`, `remove`, `findById`)

---

## 코드 스멜 제거

| 코드 스멜 | 탐지 기준 | 리팩토링 |
|-----------|----------|---------|
| Long Method | 메서드 20줄 이상 | Extract Method (원칙 1) |
| Duplicate Code | 동일/유사 코드 반복 | 메서드 추출, 템플릿 메서드 |
| Large Class | 클래스 50줄 이상 | Extract Class, SRP 적용 |
| Long Parameter List | 파라미터 3-4개 이상 | Parameter Object (원칙 3) |
| Primitive Obsession | 원시값을 도메인 개념에 직접 사용 | Value Object |
| Feature Envy | 다른 객체 데이터를 과도하게 사용 | Move Method, Tell Don't Ask |
| Data Clumps | 항상 함께 나타나는 데이터 | Extract Class, Value Object |
| Arrow Anti-Pattern | 깊이 중첩된 if/else | Early Return (원칙 2) |

---

## Validation Checklist

리팩토링 완료 후 다음을 검증하세요:

### TDD 사이클

- [ ] Red: 실패하는 테스트를 먼저 작성했는가?
- [ ] Green: 최소 구현으로 테스트를 통과했는가?
- [ ] Refactor: 테스트 통과 상태를 유지하며 리팩토링했는가?
- [ ] 모든 테스트가 통과하는가?

### Tidy First 원칙

- [ ] Structural 변경과 Behavioral 변경을 분리했는가?
- [ ] 구조적 변경 후 테스트가 여전히 통과하는가?
- [ ] 구조적 변경을 먼저 커밋했는가?

### SOLID 원칙

- [ ] SRP: 각 클래스가 하나의 책임만 가지는가?
- [ ] OCP: 확장 시 기존 코드 수정이 불필요한가?
- [ ] LSP: 하위 타입이 상위 타입을 대체 가능한가?
- [ ] ISP: 인터페이스가 작고 집중적인가?
- [ ] DIP: 구체 클래스가 아닌 추상화에 의존하는가?

### 객체지향 체조 9원칙

- [ ] 원칙 1: 한 메서드에 한 단계 들여쓰기
- [ ] 원칙 2: else 예약어 미사용 (Early Return)
- [ ] 원칙 3: 원시값/문자열 포장 (Value Object)
- [ ] 원칙 4: 한 줄에 점 하나 (디미터 법칙)
- [ ] 원칙 5: 축약 금지
- [ ] 원칙 6: 클래스 50줄 이하, 패키지 10개 이하
- [ ] 원칙 7: 인스턴스 변수 2개 이하
- [ ] 원칙 8: 일급 컬렉션 사용
- [ ] 원칙 9: Getter/Setter 최소화 (Tell, Don't Ask)

### 커밋 규율

- [ ] 모든 테스트 통과
- [ ] 컴파일/린터 경고 제로
- [ ] 커밋 메시지에 structural/behavioral 명시
- [ ] 작고 빈번한 커밋

---

## Quick Reference

### 핵심 원칙 요약

**TDD 사이클**:

1. **Red**: 실패하는 테스트 작성
2. **Green**: 최소 구현
3. **Refactor**: 아래 원칙들 적용

**Tidy First**:

- **Structural** 변경 먼저 → 별도 커밋
- **Behavioral** 변경 나중 → 별도 커밋

**객체지향 체조**:

1. **한 단계 들여쓰기** → Extract Method
2. **No else** → Early Return
3. **원시값 포장** → Value Object
4. **점 하나** → Tell, Don't Ask
5. **축약 금지** → 명확한 이름
6. **작게 유지** → 50줄/10개
7. **변수 2개 이하** → 객체 그룹화
8. **일급 컬렉션** → 비즈니스 로직 응집
9. **Tell, Don't Ask** → 객체의 자율성

---

## Additional Resources

- **TDD**: Kent Beck의 Test Driven Development: By Example
- **Tidy First**: Kent Beck의 Tidy First?
- **SOLID 원칙**: Uncle Bob Martin의 클린 아키텍처
- **객체지향 체조**: ThoughtWorks Anthology
- **DDD**: Eric Evans의 Domain-Driven Design
- **리팩토링**: Martin Fowler의 Refactoring
- **클린 코드**: Robert C. Martin의 Clean Code

---

이 스킬을 사용하여 코드를 TDD 방법론에 따라 지속적으로 개선하고, 유지보수 가능한 객체지향 코드를 작성하세요.
