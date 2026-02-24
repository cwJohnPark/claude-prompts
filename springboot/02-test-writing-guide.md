---
name: test-writing-guide
description: 테스트 코드 작성 종합 가이드. TDD/ATDD, Given-When-Then, 레이어별 테스트 전략 정의
category: springboot
tags: [spring-boot, java, testing, tdd, junit, assertj]
---

# Test Writing Guide Skill

테스트 코드 작성을 위한 종합 가이드입니다. TDD/ATDD 방법론, 테스트 구조, 네이밍 컨벤션, 레이어별 테스트 전략을 정의합니다.

## 트리거 조건

다음 키워드가 포함된 요청 시 이 스킬을 활성화합니다:

- 테스트, test, 단위테스트, 통합테스트
- TDD, ATDD, Red-Green-Refactor
- JUnit, AssertJ, MockMvc, RestAssured
- Given-When-Then
- 테스트 커버리지

---

## 1. 테스트 프레임워크 및 도구

### 1.1 핵심 의존성

```groovy
// JUnit 5 (Jupiter)
testImplementation 'org.springframework.boot:spring-boot-starter-test'

// AssertJ - 가독성 높은 assertion
testImplementation 'org.assertj:assertj-core'

// RestAssured - API 테스트
testImplementation 'io.rest-assured:rest-assured'
```

### 1.2 테스트 어노테이션

| 어노테이션                        | 용도         |
|------------------------------|------------|
| `@Test`                      | 테스트 메서드 지정 |
| `@DisplayName`               | 한글 테스트 설명  |
| `@BeforeEach` / `@AfterEach` | 테스트 전후 설정  |
| `@ParameterizedTest`         | 파라미터화된 테스트 |

---

## 2. 테스트 구조 (Given-When-Then)

모든 테스트는 **Given-When-Then** 패턴을 따릅니다:

```java
@Test
@DisplayName("유효한 요청으로 프로젝트를 생성할 수 있다")
void shouldCreateProjectWithValidRequest() {
    // Given: 테스트 데이터 준비
    ProjectCreateRequest request = new ProjectCreateRequest("Test Project", "PRJ-001");

    // When: 테스트 대상 실행
    Project result = projectService.create(request);

    // Then: 결과 검증
    assertThat(result.getName()).isEqualTo("Test Project");
    assertThat(result.getCode()).isEqualTo("PRJ-001");
}
```

### 2.1 Given 섹션

- 테스트에 필요한 데이터와 상태를 준비
- Mock 객체 설정
- 입력 파라미터 생성

### 2.2 When 섹션

- 테스트 대상 메서드 실행
- **단 하나의 액션만** 실행

### 2.3 Then 섹션

- AssertJ를 사용한 결과 검증
- 예외 검증 시 `assertThatThrownBy()` 사용

---

## 3. 테스트 네이밍 컨벤션

### 3.1 메서드명

```java
// 패턴: should{기대결과}() 또는 should{기대결과}When{조건}()
void shouldReturnEmptyListWhenNoItemsExist()
void shouldThrowExceptionWhenIdIsNull()
void shouldCalculateScoreCorrectly()
```

### 3.2 @DisplayName (한글)

```java
@DisplayName("ID가 null이면 예외를 발생시킨다")
void shouldThrowExceptionWhenIdIsNull()

@DisplayName("삭제되지 않은 항목만 조회된다")
void shouldFindOnlyNonDeletedItems()
```

### 3.3 테스트 클래스 파일 분리

테스트 대상 클래스의 각 메서드별로 **별도의 테스트 클래스 파일**을 생성합니다.

**파일 명명 규칙**: `{ClassName}_{MethodName}_Test.java`

```
src/test/java/com/example/core/domain/project/
├── ProjectService_create_Test.java
├── ProjectService_findById_Test.java
└── ProjectService_delete_Test.java
```

```java
// ProjectService_create_Test.java
@DisplayName("ProjectService.create()")
class ProjectService_create_Test {

    @Test
    @DisplayName("유효한 요청으로 프로젝트를 생성할 수 있다")
    void shouldCreateProject() { ... }

    @Test
    @DisplayName("코드가 중복되면 예외를 발생시킨다")
    void shouldThrowWhenCodeDuplicated() { ... }
}

// ProjectService_findById_Test.java
@DisplayName("ProjectService.findById()")
class ProjectService_findById_Test {

    @Test
    @DisplayName("존재하는 ID로 프로젝트를 조회할 수 있다")
    void shouldFindProjectById() { ... }
}
```

---

## 4. 레이어별 테스트 전략

### 4.1 Domain Layer (단위 테스트)

**목표**: 100% 테스트 커버리지

```java
class ScoreCalculatorTest {

    @Test
    @DisplayName("규정에 따라 점수를 계산한다")
    void shouldCalculateScoreAccordingToRegulation() {
        // Given
        BigDecimal resourceUsage = BigDecimal.valueOf(1000);
        BigDecimal distance = BigDecimal.valueOf(5000);

        // When
        BigDecimal score = calculator.calculate(resourceUsage, distance);

        // Then
        assertThat(score).isEqualByComparingTo("0.200");
    }
}
```

**특징**:

- 외부 의존성 없이 순수 Java 테스트
- 빠른 실행 속도
- Test-First 접근 권장

### 4.2 Repository Layer (통합 테스트)

**Base Class**: `AbstractRepositoryTest` 또는 `@DataJpaTest`

```java
@DataJpaTest
@ActiveProfiles("test")
@Import(CoreJpaDataSourceConfig.class)
@DisplayName("OrderRepository JPA 통합 테스트")
class OrderRepositoryTest {

    @Autowired
    private OrderRepository repository;

    @Test
    @DisplayName("메타데이터와 함께 주문을 저장할 수 있다")
    void shouldSaveWithMetadata() {
        // Given
        Order order = Order.builder()
            .userId(100L)
            .type(OrderType.STANDARD)
            .build();

        // When
        Order saved = repository.save(order);

        // Then
        assertThat(saved.getId()).isNotNull();
    }
}
```

**특징**:

- 실제 DB (H2 또는 TestContainers) 사용
- JPA 매핑 및 쿼리 검증
- `@Sql` 어노테이션으로 테스트 데이터 로드 가능

### 4.3 Application Layer (서비스 테스트)

```java
@ExtendWith(MockitoExtension.class)
class ProjectServiceTest {

    @Mock
    private ProjectRepository projectRepository;

    @InjectMocks
    private ProjectService projectService;

    @Test
    @DisplayName("프로젝트를 저장하고 반환한다")
    void shouldSaveAndReturnProject() {
        // Given
        Project project = Project.builder().name("Test").build();
        when(projectRepository.save(any())).thenReturn(project);

        // When
        Project result = projectService.create(project);

        // Then
        assertThat(result.getName()).isEqualTo("Test");
        verify(projectRepository).save(any());
    }
}
```

### 4.4 API Layer (ATDD 통합 테스트)

**Base Class**: `ApiIntegrationTest`

```java
class ProjectControllerTest extends ApiIntegrationTest {

    @Test
    @WithNormalUser  // 일반 사용자 인증
    @DisplayName("프로젝트 목록을 조회할 수 있다")
    void shouldGetProjectList() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("/api/v1/projects")
        .then()
            .statusCode(200)
            .body("content", hasSize(greaterThan(0)));
    }

    @Test
    @WithAdminUser  // 관리자 인증
    @DisplayName("관리자는 프로젝트를 생성할 수 있다")
    void shouldCreateProjectAsAdmin() {
        ProjectCreateRequest request = new ProjectCreateRequest("New Project", "PRJ-002");

        given()
            .contentType(ContentType.JSON)
            .body(request)
        .when()
            .post("/api/v1/projects")
        .then()
            .statusCode(201)
            .body("name", equalTo("New Project"));
    }
}
```

**특징**:

- `@WithNormalUser`, `@WithAdminUser` 어노테이션으로 인증 처리
- RestAssured의 fluent API 사용
- 실제 서버 실행 (`@SpringBootTest(webEnvironment = RANDOM_PORT)`)
- `@Sql`로 테스트 데이터 준비

---

## 5. ApiIntegrationTest Base Class

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@ExtendWith(WithUserExtension.class)
@Sql(scripts = {
    "classpath:sql/cleanup.sql",
    "classpath:sql/test-data.sql"
}, executionPhase = BEFORE_TEST_CLASS)
public abstract class ApiIntegrationTest {

    @LocalServerPort
    protected int port;

    @Autowired
    protected ObjectMapper objectMapper;

    @Autowired
    protected TokenService tokenService;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
    }

    // 토큰 생성 유틸리티
    protected String generateToken(String loginId, UserRole role) {
        return tokenService.generateToken(loginId, role);
    }

    protected String bearer(String token) {
        return "Bearer " + token;
    }
}
```

### 5.1 인증 어노테이션

| 어노테이션                      | 역할      | 권한         |
|----------------------------|---------|------------|
| `@WithNormalUser`          | 일반 사용자  | ROLE_USER  |
| `@WithAdminUser`           | 관리자     | ROLE_ADMIN |
| `@WithUser(loginId, role)` | 커스텀 사용자 | 지정된 역할     |

### 5.2 테스트 데이터 로드

```java
@Sql(scripts = "classpath:sql/projects-test-data.sql", executionPhase = BEFORE_TEST_METHOD)
class ProjectControllerTest extends ApiIntegrationTest {
    // ...
}
```

---

## 6. AssertJ 사용 가이드

### 6.1 기본 Assertion

```java
// 동등성 검증
assertThat(actual).isEqualTo(expected);
assertThat(actual).isNotEqualTo(unexpected);

// Null 검증
assertThat(result).isNotNull();
assertThat(optional).isPresent();
assertThat(optional).isEmpty();

// 숫자 비교
assertThat(value).isGreaterThan(10);
assertThat(value).isBetween(1, 100);
assertThat(bigDecimal).isEqualByComparingTo("100.00");
```

### 6.2 컬렉션 Assertion

```java
// 크기 검증
assertThat(list).hasSize(3);
assertThat(list).isEmpty();
assertThat(list).isNotEmpty();

// 요소 포함 여부
assertThat(list).contains(element);
assertThat(list).containsExactly(a, b, c);
assertThat(list).containsOnly(false);

// 속성 추출 및 검증
assertThat(items)
    .extracting(Item::getName)
    .containsExactly("Item A", "Item B");

assertThat(attachments)
    .extracting(Attachment::getId)
    .doesNotContainNull();
```

### 6.3 예외 Assertion

```java
// 예외 발생 검증
assertThatThrownBy(() -> service.process(null))
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessageContaining("필수입니다");

// 예외 미발생 검증
assertThatCode(() -> service.validate(validInput))
    .doesNotThrowAnyException();
```

### 6.4 문자열 Assertion

```java
assertThat(filename).startsWith("Test_Report_");
assertThat(filename).endsWith(".pdf");
assertThat(path).contains("/uploads/");
assertThat(name).isEqualTo("expected");
```

---

## 7. TDD (Test-Driven Development)

### 7.1 Red-Green-Refactor 사이클

```
1. RED: 실패하는 테스트 작성
   - 컴파일 오류 허용 (테스트 스케치)
   - 의도한 동작을 테스트로 표현

2. GREEN: 테스트 통과를 위한 최소 코드 작성
   - 가장 간단한 구현
   - 완벽하지 않아도 됨

3. REFACTOR: 코드 개선
   - 중복 제거
   - 명확한 네이밍
   - 테스트는 계속 통과해야 함
```

### 7.2 테스트 스케치 (컴파일 오류 허용)

```java
@Test
@DisplayName("점수를 계산한다")
void shouldCalculateScore() {
    // 아직 존재하지 않는 클래스/메서드 사용
    ScoreCalculator calculator = new ScoreCalculator();  // 컴파일 오류
    BigDecimal result = calculator.calculate(input, weight);  // 컴파일 오류

    assertThat(result).isEqualByComparingTo("0.200");
}
```

---

## 8. 테스트 실행 명령어

### 8.1 Gradle 명령어

```bash
# 전체 테스트 실행
./gradlew test

# 특정 모듈 테스트
./gradlew :core:test
./gradlew :app-api:test

# 특정 테스트 클래스 실행
./gradlew :core:test --tests "ProjectServiceTest"

# 특정 테스트 메서드 실행
./gradlew :core:test --tests "ProjectServiceTest.shouldCreateProject"

# 테스트 캐시 무시하고 재실행
./gradlew :core:cleanTest :core:test

# 실패한 테스트만 재실행
./gradlew test --rerun-tasks
```

### 8.2 테스트 리포트

테스트 실행 후 리포트 위치:

```
build/reports/tests/test/index.html
```

---

## 9. 테스트 커버리지 목표

| 레이어        | 목표 커버리지 | 테스트 유형      |
|------------|---------|-------------|
| Domain     | 100%    | 단위 테스트      |
| Repository | 주요 쿼리   | 통합 테스트      |
| Service    | 비즈니스 로직 | 단위/통합 테스트   |
| Controller | 주요 API  | ATDD 통합 테스트 |

---

## 10. 테스트 작성 체크리스트

### 10.1 테스트 작성 전

- [ ] 테스트할 기능/메서드의 요구사항 이해
- [ ] 정상 케이스와 예외 케이스 식별
- [ ] 경계값(boundary) 케이스 식별

### 10.2 테스트 작성 중

- [ ] Given-When-Then 구조 준수
- [ ] `@DisplayName`에 한글로 명확한 설명 작성
- [ ] 하나의 테스트에 하나의 검증 포인트
- [ ] AssertJ 사용

### 10.3 테스트 작성 후

- [ ] 테스트 실행 및 통과 확인
- [ ] 테스트 독립성 확인 (순서 무관하게 실행 가능)
- [ ] 불필요한 코드 제거

---

## 11. 안티 패턴

### 11.1 피해야 할 패턴

```java
// 여러 기능을 하나의 테스트에서 검증
@Test
void testEverything() {
    // create, update, delete 모두 테스트
}

// 테스트 간 의존성
@Test
void test1() { /* test2에서 사용할 데이터 생성 */ }
@Test
void test2() { /* test1에서 생성된 데이터 사용 */ }

// 불명확한 테스트명
@Test
void test1() { }

// 하드코딩된 sleep
Thread.sleep(1000);
```

### 11.2 권장 패턴

```java
// 하나의 테스트에 하나의 검증
@Test
@DisplayName("프로젝트를 생성할 수 있다")
void shouldCreateProject() { ... }

@Test
@DisplayName("프로젝트를 수정할 수 있다")
void shouldUpdateProject() { ... }

// 독립적인 테스트 데이터
@BeforeEach
void setUp() {
    project = createTestProject();
}

// Awaitility 사용 (비동기 테스트)
await().atMost(5, SECONDS).until(() -> result.isReady());
```

---

## 12. 실제 테스트 예시

### 12.1 Domain 단위 테스트

메서드별로 별도 테스트 파일을 생성합니다:

```
src/test/java/com/example/core/domain/attachment/
├── Attachment_of_Test.java
├── Attachment_delete_Test.java
└── Attachment_update_Test.java
```

```java
// Attachment_of_Test.java
@DisplayName("Attachment.of()")
class Attachment_of_Test {

    private static final Long OWNER_ID = 100L;
    private static final String OWNER_NAME = "Test Owner";

    @Test
    @DisplayName("첨부파일을 생성할 수 있다")
    void shouldCreateAttachment() {
        // Given
        FileMetadata metadata = createTestMetadata("document.pdf");
        AttachmentType attachmentType = AttachmentType.REPORT;

        // When
        Attachment attachment = Attachment.of(
            null, OWNER_ID, OWNER_NAME, attachmentType, LocalDate.now(), 1, metadata
        );

        // Then
        assertThat(attachment.getId()).isNull();
        assertThat(attachment.getOwnerId()).isEqualTo(OWNER_ID);
        assertThat(attachment.getAttachmentType()).isEqualTo(attachmentType);
        assertThat(attachment.isDeleted()).isFalse();
    }

    @Test
    @DisplayName("metadata가 null이면 예외를 발생시킨다")
    void shouldThrowException_whenMetadataIsNull() {
        assertThatThrownBy(() -> Attachment.of(
            null, OWNER_ID, OWNER_NAME, AttachmentType.REPORT, LocalDate.now(), 1, null
        ))
            .isInstanceOf(IllegalArgumentException.class);
    }
}
```

### 12.2 Repository 통합 테스트

```java
@DataJpaTest
@ActiveProfiles("test")
@Import(CoreJpaDataSourceConfig.class)
@DisplayName("OrderRepository JPA 통합 테스트")
class OrderRepositoryTest {

    @Autowired
    private OrderRepository repository;

    @Test
    @DisplayName("userId로 주문 목록을 조회할 수 있다")
    void shouldFindAllByUserId() {
        // Given
        Long userId = 200L;
        Order order1 = createOrder(userId, false);
        Order order2 = createOrder(userId, false);
        Order deletedOrder = createOrder(userId, true);
        repository.saveAll(List.of(order1, order2, deletedOrder));

        // When
        List<Order> results = repository.findAllByUserId(userId);

        // Then
        assertThat(results).hasSize(2);
        assertThat(results)
            .extracting(Order::isDelete)
            .containsOnly(false);
    }
}
```

### 12.3 API 통합 테스트

```java
class AttachmentsControllerTest extends ApiIntegrationTest {

    @Test
    @WithNormalUser
    @DisplayName("첨부파일 목록을 조회할 수 있다")
    void shouldGetAttachmentList() {
        given()
            .contentType(ContentType.JSON)
            .param("userId", 100L)
            .param("attachmentType", "SURVEY_REPORT")
        .when()
            .get("/api/v1/attachments")
        .then()
            .statusCode(200)
            .body("items", hasSize(greaterThanOrEqualTo(0)));
    }

    @Test
    @DisplayName("인증 없이 접근하면 401을 반환한다")
    void shouldReturn401WithoutAuthentication() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("/api/v1/attachments")
        .then()
            .statusCode(401);
    }
}
```
