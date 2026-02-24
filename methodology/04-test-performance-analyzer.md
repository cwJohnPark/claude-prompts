---
name: test-performance-analyzer
description: 테스트 실행 시간 분석 및 최적화 스킬 - 느린 테스트 원인 분석과 개선 방안 제시
category: methodology
tags: [methodology, testing, performance, optimization]
---

# Test Performance Analyzer Skill

테스트 실행 시간이 느린 원인을 분석하고 최적화 방안을 제시하는 스킬입니다.

## 트리거 조건

다음 키워드가 포함된 요청 시 이 스킬을 활성화합니다:

- 테스트 느림, 테스트 속도, 테스트 시간
- slow test, test performance
- 테스트 최적화, test optimization

---

## 1. 테스트 실행 시간 분석

### 1.1 전체 테스트 시간 측정

```bash
# 전체 테스트 실행 시간 측정
time ./gradlew test --quiet

# 모듈별 테스트 실행 시간
time ./gradlew :core:test --quiet
time ./gradlew :app-api:test --quiet
```

### 1.2 개별 테스트 실행 시간 확인

```bash
# 테스트 리포트에서 느린 테스트 확인
# 위치: build/reports/tests/test/index.html

# 특정 테스트 클래스 시간 측정
time ./gradlew :core:test --tests "ProductTest" --quiet
```

---

## 2. 느린 테스트의 주요 원인

### 2.1 Application Context 로딩 (가장 흔한 원인)

**증상**: 통합 테스트가 느림

**해결 방안**:
```java
// 느림: 전체 Context 로딩
@SpringBootTest
class SlowTest { }

// 빠름: 필요한 슬라이스만 로딩
@WebMvcTest(ProductController.class)
class FastControllerTest { }

@DataJpaTest
class FastRepositoryTest { }
```

### 2.2 데이터베이스 작업

**증상**: Repository 테스트가 느림

**해결 방안**:
- 인메모리 DB 사용
- `@Transactional`로 테스트 후 자동 롤백
- 테스트 간 데이터 격리

### 2.3 외부 서비스 호출

**증상**: HTTP 클라이언트, API 호출 테스트가 느림

**해결 방안**: Mock 서버 또는 수동 Fake 사용

### 2.4 Thread.sleep() 사용

**증상**: 특정 테스트에서 고정 대기 시간

**해결 방안**:
```java
// 느림: 하드코딩된 sleep
Thread.sleep(5000);

// 빠름: Awaitility 사용
await()
    .atMost(5, SECONDS)
    .pollInterval(100, MILLISECONDS)
    .until(() -> result.isReady());
```

### 2.5 테스트 데이터 크기

**증상**: 대량 데이터 처리 테스트가 느림

**해결 방안**:
```java
// 느림: 불필요하게 큰 데이터
List<Product> products = createProducts(10000);

// 빠름: 최소한의 데이터
List<Product> products = createProducts(3);  // 경계값만 테스트
```

### 2.6 N+1 문제

**증상**: 연관 엔티티 조회 시 느림

**해결 방안**: Fetch Join 또는 EntityGraph 사용

---

## 3. 테스트 최적화 전략

### 3.1 테스트 분류 및 병렬 실행

```java
// 테스트 태깅
@Tag("fast")
class FastUnitTest { }

@Tag("slow")
@Tag("integration")
class SlowIntegrationTest { }
```

### 3.2 Context Caching 활용

동일한 설정의 테스트는 Context를 공유하도록 구성

### 3.3 테스트 레이어 분리

| 레이어 | 속도 | 테스트 유형 | 권장 비율 |
|--------|------|-------------|-----------|
| Unit | 매우 빠름 | 순수 단위 테스트 | 70% |
| Slice | 빠름 | 슬라이스 테스트 | 20% |
| Integration | 느림 | 통합 테스트 | 10% |

---

## 4. 분석 체크리스트

### 빠른 진단

- [ ] 통합 테스트 사용 개수 확인
- [ ] `Thread.sleep()` 사용 여부 확인
- [ ] 외부 API 호출 여부 확인
- [ ] 대용량 테스트 데이터 사용 여부 확인

### 심층 분석

- [ ] 테스트 리포트에서 느린 테스트 식별
- [ ] 쿼리 통계 확인
- [ ] Context 로딩 횟수 확인
- [ ] 병렬 실행 가능 여부 확인

### 최적화 적용

- [ ] Unit 테스트로 전환 가능한 테스트 식별
- [ ] Slice 테스트로 전환 가능한 테스트 식별
- [ ] Mock/Stub 적용 가능한 외부 의존성 식별
- [ ] 테스트 데이터 최소화
