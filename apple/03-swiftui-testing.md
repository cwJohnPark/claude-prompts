---
name: swiftui-testing
description: SwiftUI 앱 테스트 전략, Unit/Integration/UI 테스트 작성 가이드
category: apple
tags: [swiftui, ios, testing, xctest, unit-test]
---

# SwiftUI 테스트 스킬

## 개요
SwiftUI 앱의 테스트 전략, 구조, 작성 방법을 정의합니다.

---

## 테스트 구조

```
Tests/
├── UnitTests/
│   ├── ViewModels/
│   │   ├── HomeViewModelTests.swift
│   │   └── SettingsViewModelTests.swift
│   ├── Services/
│   │   ├── NetworkServiceTests.swift
│   │   └── AuthServiceTests.swift
│   ├── Repositories/
│   │   └── UserRepositoryTests.swift
│   └── Models/
│       └── UserTests.swift
│
├── IntegrationTests/
│   ├── APIIntegrationTests.swift
│   └── DatabaseIntegrationTests.swift
│
└── UITests/
    ├── HomeUITests.swift
    └── SettingsUITests.swift
```

---

## 테스트 네이밍 규칙

### 형식
```
test_[테스트대상]_[시나리오]_[예상결과]
```

### 예시
```swift
// 명확한 네이밍
func test_fetchUsers_성공_사용자목록반환() { }
func test_fetchUsers_네트워크오류_에러상태설정() { }
func test_signIn_유효한자격증명_인증성공() { }
func test_signIn_잘못된비밀번호_인증실패() { }

// 불명확한 네이밍 - 피해야 함
func testFetch() { }
func testError() { }
```

---

## Unit Test

### ViewModel 테스트
```swift
import XCTest
@testable import MyApp

@MainActor
final class HomeViewModelTests: XCTestCase {

    // MARK: - Properties

    private var sut: HomeViewModel!  // System Under Test
    private var mockRepository: MockUserRepository!

    // MARK: - Setup & Teardown

    override func setUp() async throws {
        try await super.setUp()
        mockRepository = MockUserRepository()
        sut = HomeViewModel(userRepository: mockRepository)
    }

    override func tearDown() async throws {
        sut = nil
        mockRepository = nil
        try await super.tearDown()
    }

    // MARK: - Tests

    func test_loadData_성공_사용자목록설정() async {
        // Given - 테스트 조건 설정
        let expectedUsers = [
            User(id: UUID(), name: "홍길동", email: "hong@test.com", createdAt: Date()),
            User(id: UUID(), name: "김철수", email: "kim@test.com", createdAt: Date())
        ]
        mockRepository.fetchAllResult = .success(expectedUsers)

        // When - 테스트 대상 실행
        await sut.loadData()

        // Then - 결과 검증
        XCTAssertEqual(sut.users.count, 2)
        XCTAssertEqual(sut.state, .loaded)
        XCTAssertFalse(sut.showError)
    }

    func test_loadData_실패_에러상태설정() async {
        // Given
        mockRepository.fetchAllResult = .failure(TestError.networkError)

        // When
        await sut.loadData()

        // Then
        XCTAssertEqual(sut.state, .error)
        XCTAssertTrue(sut.showError)
        XCTAssertFalse(sut.errorMessage.isEmpty)
    }

    func test_loadData_이미로딩중_중복호출무시() async {
        // Given
        mockRepository.fetchAllDelay = 0.5  // 지연 설정

        // When - 첫 번째 호출
        Task { await sut.loadData() }

        // 짧은 대기 후 두 번째 호출
        try? await Task.sleep(nanoseconds: 100_000_000)
        await sut.loadData()

        // Then - 한 번만 호출되어야 함
        XCTAssertEqual(mockRepository.fetchAllCallCount, 1)
    }
}
```

### Mock 객체 구현
```swift
// MARK: - Mock Repository

final class MockUserRepository: UserRepositoryProtocol {

    // 호출 추적
    var fetchAllCallCount = 0
    var fetchCallCount = 0
    var saveCallCount = 0

    // 결과 설정
    var fetchAllResult: Result<[User], Error> = .success([])
    var fetchResult: Result<User, Error>?
    var fetchAllDelay: TimeInterval = 0

    func fetchAll() async throws -> [User] {
        fetchAllCallCount += 1

        if fetchAllDelay > 0 {
            try await Task.sleep(nanoseconds: UInt64(fetchAllDelay * 1_000_000_000))
        }

        switch fetchAllResult {
        case .success(let users):
            return users
        case .failure(let error):
            throw error
        }
    }

    func fetch(id: UUID) async throws -> User {
        fetchCallCount += 1
        guard let result = fetchResult else {
            throw TestError.notConfigured
        }

        switch result {
        case .success(let user):
            return user
        case .failure(let error):
            throw error
        }
    }

    func save(_ user: User) async throws {
        saveCallCount += 1
    }

    func delete(id: UUID) async throws { }
}

// MARK: - Test Errors

enum TestError: Error {
    case networkError
    case notConfigured
    case invalidData
}
```

### Service 테스트
```swift
import XCTest
@testable import MyApp

final class NetworkServiceTests: XCTestCase {

    private var sut: NetworkService!
    private var mockSession: MockURLSession!

    override func setUp() {
        super.setUp()
        mockSession = MockURLSession()
        sut = NetworkService(session: mockSession)
    }

    func test_request_성공_디코딩된데이터반환() async throws {
        // Given
        let expectedData = """
        [{"id": "123", "name": "테스트", "email": "test@test.com", "createdAt": "2024-01-01T00:00:00Z"}]
        """.data(using: .utf8)!

        mockSession.data = expectedData
        mockSession.response = HTTPURLResponse(
            url: URL(string: "https://api.test.com")!,
            statusCode: 200,
            httpVersion: nil,
            headerFields: nil
        )

        // When
        let users: [UserDTO] = try await sut.request(.users)

        // Then
        XCTAssertEqual(users.count, 1)
        XCTAssertEqual(users.first?.name, "테스트")
    }

    func test_request_서버오류_HTTPError발생() async {
        // Given
        mockSession.data = Data()
        mockSession.response = HTTPURLResponse(
            url: URL(string: "https://api.test.com")!,
            statusCode: 500,
            httpVersion: nil,
            headerFields: nil
        )

        // When/Then
        do {
            let _: [UserDTO] = try await sut.request(.users)
            XCTFail("오류가 발생해야 합니다")
        } catch let error as NetworkError {
            XCTAssertEqual(error, .httpError(statusCode: 500))
        } catch {
            XCTFail("NetworkError 타입이어야 합니다")
        }
    }
}
```

---

## Integration Test

### API 통합 테스트
```swift
import XCTest
@testable import MyApp

final class APIIntegrationTests: XCTestCase {

    private var networkService: NetworkService!

    override func setUp() {
        super.setUp()
        // 실제 네트워크 서비스 사용 (테스트 서버 대상)
        networkService = NetworkService()
    }

    func test_사용자API_전체흐름() async throws {
        // 이 테스트는 실제 테스트 서버와 통신합니다
        // CI/CD에서는 테스트 서버가 필요합니다

        // Given - 테스트 환경 확인
        guard ProcessInfo.processInfo.environment["RUN_INTEGRATION_TESTS"] == "true" else {
            throw XCTSkip("통합 테스트 환경이 아닙니다")
        }

        // When
        let users: [UserDTO] = try await networkService.request(.users)

        // Then
        XCTAssertFalse(users.isEmpty)
    }
}
```

---

## UI Test

### 기본 UI 테스트
```swift
import XCTest

final class HomeUITests: XCTestCase {

    private var app: XCUIApplication!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false

        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }

    func test_홈화면_사용자목록표시() {
        // Given - 홈 화면으로 이동
        let homeTab = app.buttons["홈"]
        XCTAssertTrue(homeTab.waitForExistence(timeout: 5))
        homeTab.tap()

        // When - 목록 로딩 대기
        let userList = app.tables["userList"]
        XCTAssertTrue(userList.waitForExistence(timeout: 10))

        // Then - 사용자 셀 존재 확인
        let firstCell = userList.cells.firstMatch
        XCTAssertTrue(firstCell.exists)
    }

    func test_검색_결과필터링() {
        // Given
        let searchField = app.searchFields["검색"]
        XCTAssertTrue(searchField.waitForExistence(timeout: 5))

        // When
        searchField.tap()
        searchField.typeText("홍길동")

        // Then
        let filteredCell = app.tables["userList"].cells.containing(.staticText, identifier: "홍길동")
        XCTAssertTrue(filteredCell.element.waitForExistence(timeout: 5))
    }
}
```

### 접근성 식별자 설정
```swift
// View에서 접근성 식별자 설정
struct HomeView: View {
    var body: some View {
        List(users) { user in
            UserRow(user: user)
        }
        .accessibilityIdentifier("userList")
    }
}

struct UserRow: View {
    let user: User

    var body: some View {
        HStack {
            Text(user.name)
                .accessibilityIdentifier("userName_\(user.id)")
        }
    }
}
```

---

## 테스트 유틸리티

### Test Fixtures
```swift
// MARK: - Test Fixtures

enum TestFixtures {

    static func makeUser(
        id: UUID = UUID(),
        name: String = "테스트 사용자",
        email: String = "test@example.com"
    ) -> User {
        User(
            id: id,
            name: name,
            email: email,
            createdAt: Date()
        )
    }

    static func makeUsers(count: Int) -> [User] {
        (0..<count).map { index in
            makeUser(
                name: "사용자 \(index)",
                email: "user\(index)@example.com"
            )
        }
    }
}
```

### Async Test Helpers
```swift
// MARK: - Async Test Helpers

extension XCTestCase {

    /// 비동기 조건이 충족될 때까지 대기합니다.
    func waitForCondition(
        timeout: TimeInterval = 5,
        condition: @escaping () -> Bool
    ) async throws {
        let start = Date()
        while !condition() {
            if Date().timeIntervalSince(start) > timeout {
                XCTFail("조건이 시간 내에 충족되지 않았습니다")
                return
            }
            try await Task.sleep(nanoseconds: 100_000_000)  // 0.1초
        }
    }
}
```

---

## 테스트 커버리지 목표

| 계층 | 목표 커버리지 | 우선순위 |
|------|-------------|---------|
| **ViewModel** | 80% 이상 | 높음 |
| **Service** | 70% 이상 | 높음 |
| **Repository** | 70% 이상 | 중간 |
| **Model** | 60% 이상 | 중간 |
| **View** | UI 테스트로 커버 | 낮음 |

---

## CI/CD 테스트 설정

### Xcode Cloud / GitHub Actions 설정
```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3

      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_15.0.app

      - name: Run Unit Tests
        run: |
          xcodebuild test \
            -scheme MyApp \
            -destination 'platform=macOS' \
            -resultBundlePath TestResults.xcresult

      - name: Upload Test Results
        uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: TestResults.xcresult
```

---

## 참고 자료

- [XCTest Documentation](https://developer.apple.com/documentation/xctest)
- [Testing Your Apps in Xcode](https://developer.apple.com/documentation/xcode/testing-your-apps-in-xcode)
- [UI Testing in Xcode](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/09-ui_testing.html)
