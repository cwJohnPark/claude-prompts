---
name: solid-principles-checklist
description: SOLID 5대 객체지향 설계 원칙 적용 체크리스트
category: methodology
tags: [methodology, solid, oop, design-principles]
---

# SOLID 원칙 체크리스트

SOLID 원칙은 로버트 C. 마틴(Uncle Bob)이 제안한 5가지 객체지향 설계 원칙입니다. SOLID 원칙을 적용하기 위한 상세 가이드입니다.

## 목차

1. [Single Responsibility Principle (SRP)](#1-single-responsibility-principle-srp)
2. [Open/Closed Principle (OCP)](#2-openclosed-principle-ocp)
3. [Liskov Substitution Principle (LSP)](#3-liskov-substitution-principle-lsp)
4. [Interface Segregation Principle (ISP)](#4-interface-segregation-principle-isp)
5. [Dependency Inversion Principle (DIP)](#5-dependency-inversion-principle-dip)

---

## 1. Single Responsibility Principle (SRP)

**"클래스는 단 하나의 책임만 가져야 하며, 변경 이유도 하나여야 한다."**

### 체크리스트

- [ ] 클래스가 50줄 이하인가?
- [ ] 클래스명이 명확한 단일 역할을 나타내는가?
- [ ] 모든 메서드가 클래스의 주 책임과 관련 있는가?
- [ ] 클래스 변경 시 한 가지 이유만 존재하는가?
- [ ] 메서드명에 "And"가 포함되어 있지 않은가?

### 위반 사례

```java
// 여러 책임을 가진 클래스
public class ProductService {
    public void createProduct() { }
    public void updateProduct() { }
    public double calculateScore() { }
    public byte[] generatePdfReport() { }
    public void sendNotification() { }
}
```

### 개선: 책임 분리

```java
public class ProductService {
    public void createProduct() { }
    public void updateProduct() { }
}

public class ScoreCalculationService {
    public double calculateScore() { }
}

public class ProductReportService {
    public byte[] generatePdfReport() { }
}

public class ProductNotificationService {
    public void sendNotification() { }
}
```

### 객체지향 체조와의 연계

- **원칙 1**: 한 단계 들여쓰기 → SRP 준수
- **원칙 5**: 축약 금지 → 긴 이름은 많은 책임의 신호
- **원칙 6**: 50줄 이하 클래스 → SRP 강제
- **원칙 7**: 인스턴스 변수 2개 이하 → 높은 응집도

---

## 2. Open/Closed Principle (OCP)

**"확장에는 열려 있고, 수정에는 닫혀 있어야 한다."**

### 체크리스트

- [ ] 새로운 기능 추가 시 기존 코드 수정이 불필요한가?
- [ ] 추상화(인터페이스/추상 클래스)를 활용하는가?
- [ ] 전략 패턴, 템플릿 메서드 패턴을 적용했는가?
- [ ] switch-case나 if-else 대신 다형성을 사용하는가?

### 위반 사례

```java
// 새로운 제품 타입 추가 시 수정 필요
public class ScoreCalculator {
    public double calculate(Product product) {
        if (product.getType() == ProductType.ELECTRONICS) {
            return calculateForCategoryA(product);
        } else if (product.getType() == ProductType.CLOTHING) {
            return calculateForCategoryB(product);
        }
        throw new UnsupportedOperationException("지원하지 않는 제품 타입");
    }
}
```

### 개선: 전략 패턴

```java
public interface ScoreCalculationStrategy {
    double calculate(Product product);
    ProductType supportedType();
}

public class ScoreCalculator {
    private final Map<ProductType, ScoreCalculationStrategy> strategies;

    public double calculate(Product product) {
        ScoreCalculationStrategy strategy = strategies.get(product.getType());
        if (strategy == null) {
            throw new UnsupportedOperationException("지원하지 않는 제품 타입");
        }
        return strategy.calculate(product);
    }
}
```

---

## 3. Liskov Substitution Principle (LSP)

**"하위 타입은 상위 타입을 완전히 대체할 수 있어야 한다."**

### 체크리스트

- [ ] 하위 클래스가 상위 클래스의 계약(contract)을 위반하지 않는가?
- [ ] 하위 클래스의 사전 조건이 상위 클래스보다 강하지 않은가?
- [ ] 하위 클래스의 사후 조건이 상위 클래스보다 약하지 않은가?
- [ ] 하위 클래스가 상위 클래스의 불변식(invariant)을 유지하는가?
- [ ] 오버라이드한 메서드가 예상치 못한 예외를 던지지 않는가?

### 개선 패턴

- **계약 준수**: 부모 클래스의 계약을 오버라이드하지 않음
- **조합(Composition) 사용**: 상속 대신 조합으로 LSP 위반 방지

---

## 4. Interface Segregation Principle (ISP)

**"클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다."**

### 체크리스트

- [ ] 인터페이스가 작고 집중되어 있는가?
- [ ] 구현 클래스가 인터페이스의 모든 메서드를 의미있게 구현하는가?
- [ ] 불필요한 메서드를 빈 구현으로 남기지 않았는가?

### 위반 사례

```java
// 모든 기능을 하나의 인터페이스에
public interface ProductManager {
    void create(Product product);
    void update(Product product);
    void delete(Long id);
    double calculateScore(Product product);
    byte[] generateReport(Product product);
}
```

### 개선: 인터페이스 분리

```java
public interface ProductWriter {
    void create(Product product);
    void update(Product product);
    void delete(Long id);
}

public interface ScoreCalculator {
    double calculateScore(Product product);
}

public interface ReportGenerator {
    byte[] generateReport(Product product);
}
```

---

## 5. Dependency Inversion Principle (DIP)

**"구체 클래스가 아닌 추상화에 의존해야 한다."**

### 체크리스트

- [ ] 상위 모듈이 하위 모듈에 직접 의존하지 않는가?
- [ ] 모든 의존성이 추상화(인터페이스)를 통하는가?
- [ ] 생성자 주입을 사용하는가?
- [ ] 테스트에서 쉽게 Mock/Fake로 교체 가능한가?

### 위반 사례

```java
// 구체 클래스에 직접 의존
public class OrderService {
    private final MySqlOrderRepository repository = new MySqlOrderRepository();
    private final SmtpEmailService emailService = new SmtpEmailService();
}
```

### 개선: 인터페이스에 의존

```java
public class OrderService {
    private final OrderRepository repository;
    private final NotificationService notificationService;

    public OrderService(OrderRepository repository, NotificationService notificationService) {
        this.repository = repository;
        this.notificationService = notificationService;
    }
}
```

---

## 장점 요약

| 원칙 | 핵심 장점 |
|------|----------|
| SRP | 이해 용이, 테스트 용이, 변경 용이 |
| OCP | 확장 용이, 기존 코드 안정, 유지보수 용이 |
| LSP | 안정성, 다형성, 예측 가능한 동작 |
| ISP | 낮은 결합도, 높은 응집도, 재사용성 |
| DIP | 유연성, 테스트 용이, 모듈 교체 용이 |
