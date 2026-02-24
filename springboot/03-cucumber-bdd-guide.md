---
name: cucumber-bdd-guide
description: Spring Boot Cucumber BDD 테스트 작성 가이드. Feature 파일, Step Definitions, 한국어 Gherkin 문법
category: springboot
tags: [spring-boot, java, cucumber, bdd, testing, gherkin]
---

# Cucumber BDD Guide

Spring Boot 프로젝트에서 Cucumber BDD 테스트를 작성하기 위한 가이드입니다.

## 트리거 조건

다음 키워드가 포함된 요청 시 이 스킬을 활성화합니다:

- Cucumber, BDD, Gherkin, Feature, Scenario
- 인수 테스트, Acceptance Test
- .feature 파일
- 시나리오 테스트

---

## 1. Feature 파일 작성법

### 1.1 한국어 Gherkin 문법

```gherkin
# language: ko
# src/test/resources/features/product/product-api.feature

기능: Product API
  관리자로서
  제품 센서 데이터를 관리하고 싶다
  정확한 자원 사용량을 추적하기 위해

  배경:
    조건 사용자가 "admin"으로 인증되어 있다

  @smoke @product
  시나리오: 제품 ID와 보고일자로 센서 데이터 조회
    조건 ID가 100인 제품이 존재한다
    그리고 "2024-01-15" 날짜의 보고서가 존재한다
    만약 제품 100의 "2024-01-15" 센서 데이터를 요청하면
    그러면 응답 상태 코드는 200이어야 한다
    그리고 응답에 센서 측정값이 포함되어야 한다

  시나리오 개요: 여러 날짜의 센서 데이터 조회
    조건 ID가 <productId>인 제품이 존재한다
    만약 제품 <productId>의 "<reportDate>" 센서 데이터를 요청하면
    그러면 응답 상태 코드는 <statusCode>이어야 한다

    예:
      | productId | reportDate | statusCode |
      | 100       | 2024-01-15 | 200        |
      | 999       | 2024-01-15 | 404        |
```

### 1.2 Gherkin 키워드 참조표

| 영문               | 한국어     | 용도               |
|------------------|---------|------------------|
| Feature          | 기능      | 기능/시나리오 그룹 정의    |
| Background       | 배경      | 모든 시나리오 공통 전제 조건 |
| Scenario         | 시나리오    | 개별 테스트 시나리오      |
| Scenario Outline | 시나리오 개요 | 파라미터화된 시나리오      |
| Examples         | 예       | 시나리오 개요의 데이터 테이블 |
| Given            | 조건      | 초기 상태/조건 설정      |
| When             | 만약      | 테스트 대상 액션 실행     |
| Then             | 그러면     | 결과 검증            |
| And              | 그리고     | 이전 키워드 연장        |
| But              | 하지만     | 이전 키워드의 부정적 연장   |

---

## 2. Step Definitions 작성법

### 2.1 공통 Step Definitions

```java
package com.example.app.bdd.steps;

import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.ko.그러면;
import io.cucumber.java.ko.조건;
import com.example.app.bdd.context.TestContext;

import org.springframework.beans.factory.annotation.Autowired;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * 공통 Step Definitions
 * 인증, 응답 검증 등 모든 Feature에서 재사용되는 단계를 정의합니다.
 */
public class CommonStepDefinitions {

	@Autowired
	private TestContext testContext;

	// === 영문 Step Definitions ===

	@Given("the user is authenticated as {string}")
	public void theUserIsAuthenticatedAs(String role) {
		// 인증 토큰 생성 및 저장
		testContext.setAuthToken(generateToken(role));
	}

	@Then("the response status code should be {int}")
	public void theResponseStatusCodeShouldBe(int expectedStatusCode) {
		assertThat(testContext.getLastResponse().getStatusCode())
			.isEqualTo(expectedStatusCode);
	}

	// === 한국어 Step Definitions ===

	@조건("사용자가 {string}으로 인증되어 있다")
	public void 사용자가_역할로_인증되어_있다(String role) {
		theUserIsAuthenticatedAs(role);
	}

	@그러면("응답 상태 코드는 {int}이어야 한다")
	public void 응답_상태_코드는_이어야_한다(int expectedStatusCode) {
		theResponseStatusCodeShouldBe(expectedStatusCode);
	}
}
```

### 2.2 도메인별 Step Definitions

```java
package com.example.app.bdd.steps;

import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import io.cucumber.java.ko.그러면;
import io.cucumber.java.ko.만약;
import io.cucumber.java.ko.조건;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import com.example.app.bdd.context.TestContext;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;

import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.notNullValue;

/**
 * Product Sensor API Step Definitions
 */
public class ProductSensorStepDefinitions {

	@Autowired
	private TestContext testContext;

	// === 영문 Step Definitions ===

	@Given("a product with ID {long} exists")
	public void aProductWithIdExists(Long productId) {
		// 테스트 데이터는 @Sql로 사전 로드됨
	}

	@When("I request sensor data for product {long} on {string}")
	public void iRequestSensorDataForProductOnDate(Long productId, String reportDate) {
		Response response = given()
			.header(HttpHeaders.AUTHORIZATION, testContext.bearerToken())
			.contentType(ContentType.JSON)
			.queryParam("productId", productId)
			.queryParam("reportDateTime", reportDate + "T00:00:00")
			.when()
			.get("/api/v1/admin/sensors");

		testContext.setLastResponse(response);
	}

	@Then("the response should contain sensor readings")
	public void theResponseShouldContainSensorReadings() {
		testContext.getLastResponse()
			.then()
			.body("productCode", notNullValue());
	}

	// === 한국어 Step Definitions ===

	@조건("ID가 {long}인 제품이 존재한다")
	public void id가_인_제품이_존재한다(Long productId) {
		aProductWithIdExists(productId);
	}

	@만약("제품 {long}의 {string} 센서 데이터를 요청하면")
	public void 제품의_센서_데이터를_요청하면(Long productId, String reportDate) {
		iRequestSensorDataForProductOnDate(productId, reportDate);
	}

	@그러면("응답에 센서 측정값이 포함되어야 한다")
	public void 응답에_센서_측정값이_포함되어야_한다() {
		theResponseShouldContainSensorReadings();
	}
}
```

---

## 3. 안티패턴 및 권장 패턴

### 3.1 피해야 할 패턴

```gherkin
# 너무 기술적인 시나리오
Scenario: Test API endpoint
  Given I send a GET request to "/api/v1/products/100"
  And I set header "Authorization" to "Bearer token123"
  Then I should receive HTTP status 200
  And the JSON response should have "$.name" equal to "Test Product"

# 구현 세부사항 노출
Scenario: Database insert
  Given I insert a row into products table
  When I query the products table
  Then I should get 1 row
```

### 3.2 권장 패턴

```gherkin
# 비즈니스 언어 사용
시나리오: 관리자가 제품 정보를 조회한다
  조건 관리자로 로그인되어 있다
  그리고 "테스트 제품"이 등록되어 있다
  만약 "테스트 제품" 상세 정보를 요청한다
  그러면 제품 정보가 정상적으로 반환된다

# 행동 중심 (Behavior-focused)
시나리오: 자원 사용량 리포트 생성
  조건 관리자로 인증되어 있다
  그리고 이번 달 운영 데이터가 존재한다
  만약 월간 자원 사용량 리포트를 요청하면
  그러면 항목별 자원 사용량이 포함된 리포트가 생성된다
```

### 3.3 Step 재사용성

```java
// 하드코딩된 값
@Given("the admin user is logged in")
public void adminIsLoggedIn() {
	// admin만 가능
}

// 파라미터화
@Given("the user is authenticated as {string}")
public void userIsAuthenticatedAs(String role) {
	// 모든 역할 지원
}
```

---

## 4. Feature 파일 구조화 가이드

### 4.1 Feature 파일 네이밍

```
features/
├── domain-name/
│   ├── domain-feature.feature      # 주요 기능
│   └── domain-edge-cases.feature   # 예외 케이스
├── sensor/
│   ├── sensor-api.feature
│   └── sensor-validation.feature
└── product/
    ├── product-management.feature
    └── product-search.feature
```

### 4.2 태그 전략

| 태그            | 용도                                   |
|---------------|--------------------------------------|
| `@smoke`      | 핵심 기능 빠른 검증                          |
| `@regression` | 회귀 테스트                               |
| `@wip`        | 개발 중인 시나리오                           |
| `@slow`       | 시간이 오래 걸리는 테스트                       |
| `@<domain>`   | 도메인별 분류 (예: `@user`, `@order`) |

### 4.3 Background 활용

```gherkin
기능: 주문 처리

  배경:
    조건 관리자로 인증되어 있다
    그리고 테스트 데이터가 준비되어 있다

  시나리오: 주문 상태 조회
    만약 주문 상태 조회를 요청하면
    그러면 현재 주문 상태가 반환된다

  시나리오: 주문 이력 조회
    만약 주문 이력 조회를 요청하면
    그러면 주문 이력 목록이 반환된다
```

---

## 5. JUnit vs Cucumber 사용 시점

| 상황              | 권장 도구               | 이유             |
|-----------------|---------------------|----------------|
| 단위 테스트 (Domain) | JUnit               | 빠른 피드백, 간단한 설정 |
| 기술적 통합 테스트      | JUnit + RestAssured | 개발자 친화적        |
| 비즈니스 시나리오 검증    | Cucumber            | 비즈니스 언어로 명세    |
| 인수 테스트 (ATDD)   | Cucumber            | 이해관계자와 소통 용이   |
| 복잡한 워크플로우       | Cucumber            | 시나리오 기반 문서화    |
