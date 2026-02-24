---
name: spring-api-developer
description: Spring Boot REST API 개발 표준 가이드. Controller, DTO, Swagger, 통합 테스트 작성 시 활용
category: springboot
tags: [spring-boot, java, rest-api, swagger, testing]
---

# Spring API Developer Skill

이 스킬은 Spring Boot REST API 개발 표준을 가이드합니다.

## When to Use This Skill

- REST API 엔드포인트 생성 또는 수정
- Controller, Service, Repository 레이어 작업
- DTO 작성 또는 변경
- Swagger/OpenAPI 문서화
- API 통합 테스트 작성

## API Development Workflow

### Complete Checklist

새로운 API 엔드포인트를 개발할 때 다음 순서를 따르세요:

```markdown
## API Development Checklist

- [ ]
    1. Response DTO 설계
- [ ]
    2. Request DTO 작성 (필요시)
- [ ]
    3. Service 레이어 로직 구현
- [ ]
    4. Controller 작성 (응답 래핑 확인)
- [ ]
    5. Swagger Docs 인터페이스 작성
- [ ]
    6. 통합 테스트 작성
- [ ]
    7. HTTP 클라이언트 테스트 파일 작성 (선택)
- [ ]
    8. 테스트 실행 및 검증
```

### Step-by-Step Guide

**Step 1: Design DTOs**

- 필수/선택 필드 결정
- 중첩 객체 구조 설계

**Step 2-4: Implement Business Logic**

- Service -> Repository -> Domain 레이어 구현
- Controller에서 응답 래핑 적용

**Step 5-8: Documentation & Testing**

- Swagger Docs 인터페이스 작성
- 통합 테스트로 검증

---

## Controller Guidelines

### SOLID Principles (Strictly Applied)

모든 Controller는 다음 원칙을 준수해야 합니다:

- **Single Responsibility**: Controller는 HTTP 요청/응답 처리만
- **Dependency Inversion**: Service 인터페이스에 의존
- **Interface Segregation**: 불필요한 의존성 주입 금지
- @RequestMapping 사용 금지, @GetMapping, @PostMapping 등을 사용하여, api full path를 메서드마다 입력

### Controller Structure

```java
package com.example.app.project.controller;

import lombok.RequiredArgsConstructor;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequiredArgsConstructor
public class ProjectController implements ProjectControllerDocs {

	private final ProjectService projectService;

	@GetMapping("/api/v1/projects")
	public ResponseEntity<ProjectListResponse> getProjects(
		@CurrentUser User currentUser,
		@PageableDefault(size = 20) Pageable pageable
	) {
		List<Project> projects = projectService.findByCompany(
			currentUser.getCompanyId(),
			pageable
		);
		return ResponseEntity.ok(ProjectListResponse.from(projects));
	}

	@PostMapping("/api/v1/projects")
	public ResponseEntity<ProjectResponse> createProject(
		@CurrentUser User currentUser,
		@Valid @RequestBody CreateProjectRequest request
	) {
		Project project = projectService.create(
			currentUser.getCompanyId(),
			request.toEntity()
		);
		return ResponseEntity.ok(ProjectResponse.from(project));
	}
}
```

### Critical Rules

**DO:**

- 모든 엔드포인트는 `/api/v1` 접두사 사용
- List 반환 시 반드시 wrapper 객체로 감싸기 (예: `ProjectListResponse`)
- `ResponseEntity<T>` 사용하여 응답 반환
- `@Valid` 사용 시 Controller와 Docs 인터페이스 모두에 명시
- Early return 패턴 사용 (else 절 금지)

**DON'T:**

- List를 직접 반환: `List<ProjectResponse>` (X)
- Full package name 사용 (항상 import 사용)
- else 절 사용 (early return으로 대체)

### Response Wrapping Example

```java
// BAD: List 직접 반환
@GetMapping
public ResponseEntity<List<ProjectResponse>> getProjects() {
	return ResponseEntity.ok(projectService.findAll());
}

// GOOD: Wrapper로 감싸서 반환
@GetMapping
public ResponseEntity<ProjectListResponse> getProjects() {
	List<Project> projects = projectService.findAll();
	return ResponseEntity.ok(ProjectListResponse.from(projects));
}
```

---

## DTO Standards

### Naming Conventions

- Request DTO: `{Purpose}Request` (예: `CreateProjectRequest`, `UpdateProjectRequest`)
- Response DTO: `{Purpose}Response` (예: `ProjectResponse`, `ProjectListResponse`)
- List wrapper: `{Entity}ListResponse` (예: `ProjectListResponse`)

### DTO Structure Template

```java
package com.example.app.project.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import com.example.core.domain.project.Project;
import lombok.Builder;

@Schema(description = "프로젝트 생성 요청")
@Builder
public record CreateProjectRequest(

	@Schema(description = "이름", example = "Spring Project")
	String name,

	@Schema(description = "코드", example = "PRJ-001")
	String code,

	@Schema(description = "타입", example = "STANDARD")
	ProjectType type,

	@Schema(description = "설명", example = "Sample project")
	String description
) {
	// DTO -> Entity 변환 (캡슐화 유지)
	public Project toEntity() {
		return Project.builder()
			.name(name)
			.code(code)
			.type(type)
			.description(description)
			.build();
	}
}
```

```java
@Schema(description = "프로젝트 응답")
@Builder
public record ProjectResponse(

	@Schema(description = "이름", example = "Spring Project")
	String name,

	@Schema(description = "코드", example = "PRJ-001")
	String code,

	@Schema(description = "타입", example = "STANDARD")
	ProjectType type,

	@Schema(description = "설명", example = "Sample project")
	String description
) {
	// Entity -> DTO 변환 (캡슐화 유지)
	public static ProjectResponse from(Project project) {
		return ProjectResponse.builder()
			.name(project.getName())
			.code(project.getCode())
			.type(project.getType())
			.description(project.getDescription())
			.build();
	}
}
```

```java
@Schema(description = "프로젝트 목록 응답")
public record ProjectListResponse(

	@Schema(description = "프로젝트 목록")
	List<ProjectResponse> projects,

	@Schema(description = "총 개수", example = "42")
	int totalCount
) {
	public static ProjectListResponse from(List<Project> projects) {
		List<ProjectResponse> projectResponses = projects.stream()
			.map(ProjectResponse::from)
			.toList();
		return new ProjectListResponse(projectResponses, projectResponses.size());
	}
}
```

### DTO Rules

**DO:**

- 모든 필드에 `@Schema` 어노테이션 추가 (description + example)
- 클래스 레벨에도 `@Schema(description)` 추가
- DTO <-> Entity 변환 메서드를 DTO 내부에 작성 (캡슐화)
- Record 또는 5개 이상의 인자값일 경우 `@Builder` 사용

**DON'T:**

- ID 필드 포함 (admin API 제외)
- Entity를 직접 반환
- 변환 로직을 Controller나 Service에 작성

---

## Swagger Documentation

### Separate Docs Interface Pattern

모든 Controller는 별도의 `*Docs` 인터페이스를 구현해야 합니다.

```java
package com.example.app.project.controller;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import com.example.app.common.annotation.PageableAsQueryParam;
import com.example.app.security.CurrentUser;

import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestBody;

@Tag(name = "프로젝트 관리", description = "프로젝트 정보 조회 및 관리 API")
public interface ProjectControllerDocs {

	@Operation(
		summary = "프로젝트 목록 조회",
		description = "회사의 프로젝트 목록을 페이징하여 조회합니다."
	)
	@PageableAsQueryParam
	ResponseEntity<ProjectListResponse> getProjects(
		@Parameter(hidden = true) @CurrentUser User currentUser,
		@Parameter(hidden = true, required = false) @PageableDefault(size = 20) Pageable pageable
	);

	@Operation(
		summary = "프로젝트 생성",
		description = "새로운 프로젝트를 등록합니다."
	)
	ResponseEntity<ProjectResponse> createProject(
		@Parameter(hidden = true) @CurrentUser User currentUser,
		@Valid @RequestBody CreateProjectRequest request
	);
}
```

### Swagger Docs Interface Rules

**DO:**

- 모든 문서는 한글로 작성
- `@CurrentUser`에 `@Parameter(hidden = true)` 필수
- Pageable에 `@Parameter(hidden = true, required = false)` 필수
- 페이징 API는 `@PageableAsQueryParam` 추가
- Controller에서 `@Valid` 사용 시 Docs에도 동일하게 명시

**DON'T:**

- `@CurrentUser`를 Swagger UI에 노출하지 말 것
- Pageable 파라미터를 문서화에 노출하지 말 것
- 응답, 요청에 대한 상세 설명을 노출하지 말 것

---

## Testing Requirements

### Test Structure (ATDD Pattern)

**모든 API 테스트는 `ApiIntegrationTest`를 상속하고 ATDD(Acceptance Test-Driven Development) 패턴을 따릅니다.**

```java
package com.example.app.project.controller;

import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.*;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;

import com.example.api.admin.users.domain.UserRole;
import com.example.api.support.ApiIntegrationTest;
import com.example.api.support.WithUser;

/**
 * 프로젝트 관리 API 통합 테스트
 * ApiIntegrationTest를 상속하여 RestAssured 자동 설정 및 인증 처리
 */
@WithUser(role = UserRole.USER)  // 클래스 레벨: 모든 테스트에 기본 USER 권한
class ProjectControllerTest extends ApiIntegrationTest {

	@Test
	@DisplayName("프로젝트 목록 조회 성공")
	void shouldGetProjectList() {
		// Given: 인증된 사용자 (클래스 레벨 @WithUser로 자동 설정)

		// When: 프로젝트 목록 조회
		given()
			.queryParam("page", 0)
			.queryParam("size", 20)
			.when()
			.get("/api/v1/projects")
			// Then: 성공 응답 및 데이터 검증
			.then()
			.statusCode(HttpStatus.OK.value())
			.body("projects", notNullValue())
			.body("totalCount", greaterThanOrEqualTo(0));
	}

	@Test
	@DisplayName("프로젝트 생성 성공")
	void shouldCreateProject() throws Exception {
		// Given: 프로젝트 생성 요청 데이터
		String requestBody = """
			{
			    "name": "Spring Project",
			    "code": "PRJ-001",
			    "type": "STANDARD",
			    "description": "Sample project"
			}
			""";

		// When: 프로젝트 생성 요청
		given()
			.contentType(MediaType.APPLICATION_JSON_VALUE)
			.body(requestBody)
			.when()
			.post("/api/v1/projects")
			// Then: 생성 성공 및 응답 데이터 검증
			.then()
			.statusCode(HttpStatus.OK.value())
			.body("name", equalTo("Spring Project"))
			.body("code", equalTo("PRJ-001"))
			.body("type", equalTo("STANDARD"));
	}

	@Test
	@WithUser(role = UserRole.ADMIN)  // 메서드 레벨: 이 테스트만 ADMIN 권한
	@DisplayName("관리자 권한으로 모든 회사 프로젝트 조회 성공")
	void shouldGetAllProjectsAsAdmin() {
		// Given: ADMIN 권한 사용자

		// When: 전체 프로젝트 조회
		given()
			.queryParam("includeAllCompanies", true)
			.when()
			.get("/api/v1/projects")
			// Then: 모든 회사의 프로젝트 조회 가능
			.then()
			.statusCode(HttpStatus.OK.value())
			.body("projects", notNullValue());
	}

	@Test
	@DisplayName("유효하지 않은 요청으로 프로젝트 생성 실패")
	void shouldFailToCreateProjectWithInvalidData() throws Exception {
		// Given: 필수 필드 누락된 요청
		String invalidRequestBody = """
			{
			    "name": ""
			}
			""";

		// When: 프로젝트 생성 요청
		given()
			.contentType(MediaType.APPLICATION_JSON_VALUE)
			.body(invalidRequestBody)
			.when()
			.post("/api/v1/projects")
			// Then: 400 Bad Request
			.then()
			.statusCode(HttpStatus.BAD_REQUEST.value());
	}
}
```

### ApiIntegrationTest 상속의 이점

**자동 제공되는 기능:**

- RestAssured 자동 설정 (port, baseURI, logging)
- `@WithUser` 어노테이션으로 인증 컨텍스트 자동 설정
- SQL 테스트 데이터 자동 로드 및 정리
- `objectMapper`, `tokenService` 자동 주입
- 헬퍼 메서드: `generateToken()`, `bearer()`, `assertAuthorized()`

**불필요한 보일러플레이트 제거:**

- `@LocalServerPort` 선언 불필요
- `@BeforeEach`에서 RestAssured 설정 불필요
- 수동 토큰 생성 불필요 (`@WithUser` 사용)
- Authorization 헤더 수동 추가 불필요 (자동 주입)

### Test Naming Conventions (ATDD Style)

**테스트 클래스:**

- 패턴: `{Subject}Test`
- 예: `ProjectControllerTest`, `UserAuthenticationTest`

**테스트 메서드:**

- 패턴: `should{ExpectedBehavior}When{Condition}()`
- 항상 `@DisplayName`으로 한글 설명 추가
- Given-When-Then 주석으로 의도 명확화

**예시:**

```java
@Test
@DisplayName("권한 없는 사용자가 관리자 API 호출 시 403 반환")
void shouldReturn403WhenUnauthorizedUserAccessesAdminApi() {
	// Given: 일반 사용자 권한
	// When: 관리자 전용 API 호출
	// Then: 403 Forbidden
}
```

### Test Coverage Goals (ATDD Pattern)

**필수 시나리오:**

- [ ] **정상 시나리오 (Happy Path)**: 모든 입력이 유효하고 권한이 있는 경우
- [ ] **인가 실패**: 권한 부족 (USER가 ADMIN API 호출)
- [ ] **유효성 검증 실패**: 필수 필드 누락, 잘못된 형식, 범위 초과
- [ ] **비즈니스 규칙 위반**: 중복 데이터, 존재하지 않는 리소스
- [ ] **엣지 케이스**: 빈 리스트, null 값, 경계값

**Given-When-Then 구조:**

```java
// Given: 테스트 전제 조건 (데이터, 권한, 상태)
// When: 실행할 동작 (API 호출)
// Then: 기대 결과 (상태 코드, 응답 본문, 부수 효과)
```

### Running Tests

```bash
# 전체 API 모듈 테스트
./gradlew :app-api:test

# 특정 테스트 클래스 실행
./gradlew :app-api:test --tests "*ProjectControllerTest"

# 특정 테스트 메서드 실행
./gradlew :app-api:test --tests "*ProjectControllerTest.shouldGetProjectList"
```

### Using ApiIntegrationTest Helper Methods

`ApiIntegrationTest`는 테스트 작성을 간편하게 하는 여러 헬퍼 메서드를 제공합니다:

```java
class CustomAuthTest extends ApiIntegrationTest {

	@Test
	@DisplayName("수동으로 생성한 토큰으로 API 호출")
	void shouldAccessApiWithManualToken() {
		// Given: 특정 사용자로 수동 토큰 생성
		String customToken = generateToken("custom@mail.com", UserRole.ADMIN);

		// When & Then
		given()
			.header("Authorization", bearer(customToken))
			.when()
			.get("/api/v1/admin/users")
			.then()
			.statusCode(200);
	}

	@Test
	@DisplayName("권한별 API 접근 테스트")
	void shouldTestAuthorizationLevels() {
		// Given: 각 권한별 토큰 생성
		String userToken = generateToken("user@mail.com", UserRole.USER);
		String adminToken = generateToken("admin@mail.com", UserRole.ADMIN);

		// When & Then: USER는 403, ADMIN은 200
		assertAuthorized(userToken, "/api/v1/admin/users", 403);
		assertAuthorized(adminToken, "/api/v1/admin/users", 200);
	}

	@Test
	@DisplayName("ObjectMapper로 복잡한 응답 파싱")
	void shouldParseComplexResponse() throws Exception {
		// Given: API 호출 후 응답 획득
		String responseBody = given()
			.when()
			.get("/api/v1/projects")
			.then()
			.statusCode(200)
			.extract()
			.asString();

		// When: ObjectMapper로 파싱 (objectMapper는 자동 주입됨)
		ProjectListResponse response = objectMapper.readValue(
			responseBody,
			ProjectListResponse.class
		);

		// Then: 파싱된 객체로 검증
		assertThat(response.projects()).isNotEmpty();
		assertThat(response.totalCount()).isGreaterThan(0);
	}
}
```

**제공되는 헬퍼 메서드:**

- `generateToken(loginId, role)`: 특정 사용자와 권한으로 JWT 토큰 생성
- `bearer(token)`: Bearer 접두사를 붙인 Authorization 헤더 값 생성
- `assertAuthorized(token, path, expectedStatus)`: 권한별 접근 테스트 간편화
- `objectMapper`: JSON 직렬화/역직렬화 (자동 주입)
- `tokenService`: 토큰 생성/검증 서비스 (자동 주입)

---

## Common Patterns

### Handling Validation Errors

```java
@PostMapping
public ResponseEntity<ProjectResponse> createProject(
	@CurrentUser User currentUser,
	@Valid @RequestBody CreateProjectRequest request
) {
	// @Valid는 자동으로 검증 오류를 처리
	// GlobalExceptionHandler에서 MethodArgumentNotValidException 처리
	Project project = projectService.create(currentUser.getCompanyId(), request.toEntity());
	return ResponseEntity.ok(ProjectResponse.from(project));
}
```

### Early Return Pattern (No else clause)

```java
// BAD: else 절 사용
public ProjectResponse getProject(Long id) {
	Project project = projectRepository.findById(id);
	if (project == null) {
		throw new ProjectNotFoundException();
	} else {
		return ProjectResponse.from(project);
	}
}

// GOOD: Early return
public ProjectResponse getProject(Long id) {
	Project project = projectRepository.findById(id);
	if (project == null) {
		throw new ProjectNotFoundException();
	}
	return ProjectResponse.from(project);
}
```

### Entity Update Pattern

```java
// Entity 내부 메서드
public class Project {

	// BAD: 개별 필드를 인자로 받음
	public void update(String name, String code, ProjectType type) {
		this.name = name;
		this.code = code;
		this.type = type;
	}

	// GOOD: 동일 타입의 entity를 매개변수로 받음
	public void update(Project other) {
		this.name = other.name;
		this.code = other.code;
		this.type = other.type;
	}
}
```

---

## Validation Checklist

개발 완료 전 다음 항목을 검증하세요:

### Controller Validation

- [ ] `/api/v1` 접두사 사용
- [ ] Docs 인터페이스 구현
- [ ] List 반환 시 wrapper 사용
- [ ] `ResponseEntity<T>` 반환
- [ ] Early return 패턴 적용
- [ ] else 절 미사용

### DTO Validation

- [ ] Request/Response 접미사 사용
- [ ] 모든 필드에 `@Schema` 추가
- [ ] description + example 명시
- [ ] ID 필드 미포함 (admin 제외)
- [ ] toEntity/from 메서드 구현

### Swagger Validation

- [ ] 별도 Docs 인터페이스 작성
- [ ] `@CurrentUser`에 `@Parameter(hidden=true)`
- [ ] Pageable에 `@Parameter(hidden=true, required=false)`
- [ ] `@PageableAsQueryParam` 추가 (페이징 시)
- [ ] 모든 문서 한글 작성

### Testing Validation

- [ ] `ApiIntegrationTest` 상속
- [ ] `@WithUser` 어노테이션으로 인증 설정
- [ ] Given-When-Then 주석 포함
- [ ] 정상/실패/권한 케이스 포함
- [ ] 테스트 메서드명이 `should{Behavior}When{Condition}` 패턴
- [ ] `@DisplayName`으로 한글 설명 추가
- [ ] 테스트 실행 성공 확인 (`./gradlew :app-api:test`)

---

## Code Review Checklist

For every controller you review, verify:
- [ ] No @RequestMapping on methods (only @GetMapping, @PostMapping, etc.)
- [ ] All requests use @Validated or @Valid
- [ ] All responses wrapped in ResponseEntity
- [ ] @CurrentUser paired with @Parameter(hidden = true)
- [ ] No business logic in controller methods

For every DTO you review, verify:
- [ ] Request DTOs have validation annotations
- [ ] Names end with Request or Response
- [ ] No id fields (unless Admin API)
- [ ] Conversion methods present (toEntity/from)

For every Swagger doc you review, verify:
- [ ] Documentation in separate *Docs interface
- [ ] @CurrentUser has @Parameter(hidden = true)
- [ ] Pagination uses @PageableAsQueryParam
- [ ] All descriptions in Korean

For every test you review, verify:
- [ ] Extends ApiIntegrationTest
- [ ] @WithUser on test methods
- [ ] No 401/403 test cases (handled separately)
- [ ] Naming follows should{Behavior}When{Condition}

---

## Quick Reference

### Module Structure

```
app-api/
└── src/main/java/com/example/app/
    └── {domain}/
        ├── controller/
        │   ├── {Domain}Controller.java
        │   └── {Domain}ControllerDocs.java
        ├── dto/
        │   ├── Create{Domain}Request.java
        │   ├── Update{Domain}Request.java
        │   ├── {Domain}Response.java
        │   └── {Domain}ListResponse.java
        └── service/
            ├── {Domain}Service.java
            └── {Domain}ServiceImpl.java
```

### Key Annotations

**Controller & API:**

- `@RestController`: Controller 클래스
- `@GetMapping`, `@PostMapping`, etc.: HTTP 메서드 매핑 (`@RequestMapping` 사용 금지)
- `@CurrentUser`: 인증된 사용자 주입
- `@Valid`: 요청 본문 검증
- `@Schema`: Swagger 문서화
- `@Parameter(hidden=true)`: Swagger에서 숨김
- `@PageableAsQueryParam`: 페이징 파라미터 문서화

**Testing:**

- `@WithUser(role = UserRole.USER)`: 테스트 사용자 권한 설정
- `@DisplayName("...")`: 테스트 설명 (한글)
- `extends ApiIntegrationTest`: 통합 테스트 베이스 클래스

### Common Commands

```bash
# API 서버 실행
./gradlew :app-api:bootRun

# 테스트 실행
./gradlew :app-api:test

# 특정 테스트 실행
./gradlew :app-api:test --tests "*ControllerTest"
```

## Communication Style

- Be direct and precise in identifying issues
- Provide complete code examples, not fragments
- Prioritize critical violations (security, architecture) over style issues
- Use Korean for comments when generating code
- Acknowledge when code follows standards correctly
- Suggest improvements even for compliant code when better patterns exist
