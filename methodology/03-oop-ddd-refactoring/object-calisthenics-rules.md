---
name: object-calisthenics-rules
description: ThoughtWorks 객체지향 체조 9원칙 적용 상세 가이드
category: methodology
tags: [methodology, oop, object-calisthenics, refactoring]
---

# 객체지향 체조 9원칙 상세 가이드

ThoughtWorks Anthology에서 제안한 객체지향 체조(Object Calisthenics) 9원칙을 적용하는 상세 가이드입니다.

## 목차

1. [한 메서드에 오직 한 단계의 들여쓰기](#원칙-1-한-메서드에-오직-한-단계의-들여쓰기)
2. [else 예약어 금지](#원칙-2-else-예약어-금지)
3. [모든 원시값과 문자열 포장](#원칙-3-모든-원시값과-문자열-포장)
4. [한 줄에 점 하나](#원칙-4-한-줄에-점-하나-디미터-법칙)
5. [축약 금지](#원칙-5-축약-금지)
6. [엔티티를 작게 유지](#원칙-6-엔티티를-작게-유지)
7. [인스턴스 변수 2개 이하](#원칙-7-인스턴스-변수-2개-이하)
8. [일급 컬렉션 사용](#원칙-8-일급-컬렉션-사용)
9. [Getter/Setter 지양](#원칙-9-gettersetter-지양)

---

## 원칙 1: 한 메서드에 오직 한 단계의 들여쓰기

### WHY?

코드의 들여쓰기 깊이가 깊어질수록 가독성이 급격히 하락합니다.

**"함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다."** - 로버트 C. 마틴, 클린 코드

### 위반 사례

```java
// BAD: 3단계 들여쓰기
public class ReportGenerator {
    public String generateReport(List<Product> products) {
        StringBuilder report = new StringBuilder();
        for (Product product : products) {
            report.append("Product: ").append(product.getName()).append("\n");
            for (DailyRecord record : product.getRecords()) {
                report.append("  Date: ").append(record.getDate()).append("\n");
                for (ResourceUsage resource : record.getResources()) {
                    report.append("    Resource: ").append(resource.getType()).append("\n");
                }
            }
        }
        return report.toString();
    }
}
```

### 해결책: Extract Method

```java
// GOOD: 한 단계 들여쓰기
public class ReportGenerator {
    public String generateReport(List<Product> products) {
        StringBuilder report = new StringBuilder();
        for (Product product : products) {
            appendProductSection(report, product);
        }
        return report.toString();
    }

    private void appendProductSection(StringBuilder report, Product product) {
        report.append("Product: ").append(product.getName()).append("\n");
        for (DailyRecord record : product.getRecords()) {
            appendRecordSection(report, record);
        }
    }

    private void appendRecordSection(StringBuilder report, DailyRecord record) {
        report.append("  Date: ").append(record.getDate()).append("\n");
        for (ResourceUsage resource : record.getResources()) {
            report.append("    Resource: ").append(resource.getType()).append("\n");
        }
    }
}
```

**관련 원칙**: SRP (Single Responsibility Principle)

---

## 원칙 2: else 예약어 금지

### WHY?

`else` 예약어는 조건 분기의 depth를 깊게 만들고 Arrow Anti-Pattern을 유발합니다.

### 해결책 1: Early Return 패턴

```java
// GOOD: Early Return
public String getGrade(double score) {
    if (score < 0.8) {
        return "A";
    }

    if (score < 0.95) {
        return "B";
    }

    if (score < 1.10) {
        return "C";
    }

    return "D";
}
```

### 해결책 2: Guard Clause (보호 절)

```java
// GOOD: Guard Clause
public void registerProduct(Product product) {
    if (product == null) {
        throw new IllegalArgumentException("제품 정보는 필수입니다");
    }

    if (product.getCode() == null) {
        throw new IllegalArgumentException("제품 코드는 필수입니다");
    }

    repository.save(product);
}
```

### 해결책 3: 다형성 활용 (전략 패턴)

```java
// GOOD: 전략 패턴으로 else 제거
public interface CostCalculator {
    double calculate(double amount);
}

public class MaterialCostCalculator implements CostCalculator {
    public double calculate(double amount) {
        return amount * 3.114;
    }
}

public class LaborCostCalculator implements CostCalculator {
    public double calculate(double amount) {
        return amount * 3.206;
    }
}
```

**관련 원칙**: OCP (전략 패턴), SRP

---

## 원칙 3: 모든 원시값과 문자열 포장

### WHY?

원시값은 아무런 의미를 가지지 않으며, 의미는 오직 변수명으로만 추론할 수 있습니다. 이는 **Primitive Obsession** 안티 패턴입니다.

### 문제점

```java
// BAD: 모든 파라미터가 동일한 타입 - 순서 혼동 가능
public class Product {
    private int productCode;
    private double weight;
    private double distance;

    public Product(int productCode, double weight, double distance) {
        this.productCode = productCode;
        this.weight = weight;
        this.distance = distance;
    }
}

Product product = new Product(75000, 9876543, 1000.0); // 순서 혼동!
```

### 해결책: Value Object 패턴

```java
// GOOD: Value Object
public class ProductCode {
    private final int value;

    public ProductCode(int value) {
        if (value < 1000000 || value > 9999999) {
            throw new IllegalArgumentException("제품 코드는 7자리여야 합니다");
        }
        this.value = value;
    }
}

public class Weight {
    private final double value;

    public Weight(double value) {
        if (value <= 0) {
            throw new IllegalArgumentException("무게는 양수여야 합니다");
        }
        this.value = value;
    }
}

public class Distance {
    private final double meters;

    public Distance(double meters) {
        if (meters < 0) {
            throw new IllegalArgumentException("거리는 음수가 될 수 없습니다");
        }
        this.meters = meters;
    }

    public double toKilometers() { return meters / 1000; }
}
```

**장점**: 타입 안전성, 유효성 검증 일원화, 비즈니스 로직 응집

**관련 원칙**: DDD Value Object, SRP

---

## 원칙 4: 한 줄에 점 하나 (디미터 법칙)

### WHY?

여러 점은 결합도를 높이고 캡슐화를 위반합니다. "낯선 이와 이야기하지 말라" (Law of Demeter)

### 위반 사례

```java
// BAD: 체이닝으로 깊은 의존성
double distance = product.getOrderDetails().getRoute().getSteps()
    .stream().mapToDouble(wp -> wp.getDistance().getMeters()).sum();
```

### 해결책: Tell, Don't Ask

```java
// GOOD: 객체에게 물어보지 말고 시켜라
double distance = product.getTotalRouteDistance();
```

**예외**: DTO, Fluent API, Stream API

---

## 원칙 5: 축약 금지

### WHY?

축약된 이름은 혼란을 야기합니다. 긴 이름은 클래스/메서드가 너무 많은 일을 한다는 신호입니다.

```java
// BAD
public class PrdSvc { }
public class PriceCalc { }

// GOOD
public class ProductService { }
public class PriceCalculator { }
```

**원칙**: 문맥 중복 제거: `Order.shipOrder()` → `Order.ship()`

---

## 원칙 6: 엔티티를 작게 유지

### 규칙

- 클래스: **50줄 이하**
- 패키지: **10개 파일 이하**

큰 클래스는 한 가지 일을 하지 않으며 이해와 재사용이 어렵습니다.

**해결**: SRP 적용하여 책임 분리

---

## 원칙 7: 인스턴스 변수 2개 이하

### WHY?

많은 인스턴스 변수는 낮은 응집도를 의미합니다.

```java
// BAD: 5개의 인스턴스 변수
public class Product {
    private String name;
    private String productCode;
    private ProductType type;
    private double price;
    private int buildYear;
}

// GOOD: 인스턴스 변수를 객체로 그룹화
public class Product {
    private final ProductIdentity identity;
    private final ProductSpecification spec;
}
```

---

## 원칙 8: 일급 컬렉션 사용

### WHY?

컬렉션을 포장하지 않으면 비즈니스 규칙이 흩어지고 중복됩니다.

```java
// GOOD: 일급 컬렉션
public class OrderItems {
    private final List<OrderItem> items;

    public void add(OrderItem item) {
        validateNoDuplicate(item);
        items.add(item);
    }

    public Money calculateSubtotal() {
        return items.stream()
            .map(OrderItem::getPrice)
            .reduce(Money.zero(), Money::add);
    }

    public int size() { return items.size(); }

    public List<OrderItem> toList() {
        return Collections.unmodifiableList(items);
    }
}
```

**장점**: 비즈니스 규칙 응집, 중복 제거, 캡슐화, 불변성

---

## 원칙 9: Getter/Setter 지양

### WHY?

Getter/Setter 남발은 객체의 자율성을 해치고 캡슐화를 위반합니다. **Tell, Don't Ask** 원칙을 따르세요.

```java
// BAD: Getter로 상태를 가져와서 외부에서 계산
double distance = order.getRoute().getTotalDistance();
double time = order.getDuration();
double speed = distance / time;

// GOOD: 객체에게 계산을 시킴
double speed = order.calculateAverageSpeed();
```

**허용되는 Getter 사용**:
- DTO/Response 객체 (데이터 전송 목적)
- Entity to DTO 변환 시
- 테스트 검증 시
- 프레임워크 요구사항 (JPA 등)
