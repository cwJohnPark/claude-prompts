---
name: swiftui-coding-rules
description: SwiftUI 앱 개발 코딩 규칙 및 SOLID 원칙 적용 가이드
category: apple
tags: [swiftui, ios, coding-standards, solid]
---

# SwiftUI 코딩 규칙 스킬

## 개요
SwiftUI 앱 개발 시 적용해야 하는 코딩 규칙과 SOLID 원칙을 정의합니다.

---

## 일반 코딩 규칙

### 코드 구조
- 중첩 로직 깊이를 피하고, 명확한 종료 조건을 가진 평탄한 구조 선호
- 가능하면 파일당 **100줄 미만**으로 작성
- 메서드는 작고 응집력 있게 유지
- Guard 문을 활용한 Early Return 패턴 적극 사용

```swift
// 중첩된 구조 - 피해야 함
func processData(_ data: Data?) {
    if let data = data {
        if data.count > 0 {
            if let result = parse(data) {
                // 처리
            }
        }
    }
}

// 평탄한 구조 - 권장
func processData(_ data: Data?) {
    guard let data = data, data.count > 0 else { return }
    guard let result = parse(data) else { return }
    // 처리
}
```

### 데이터 처리
- 불변(immutable) 데이터 구조 선호
- `struct` 우선, 필요한 경우에만 `class` 사용
- `let` 우선, 필요한 경우에만 `var` 사용

```swift
// 불변 데이터 구조 권장
struct User {
    let id: UUID
    let name: String
    let email: String
}
```

---

## SOLID 원칙 적용

| 원칙 | 설명 | SwiftUI 적용 |
|------|------|--------------|
| **단일 책임 (SRP)** | 각 클래스나 메서드는 하나의 역할만 수행 | View는 표시만, ViewModel은 로직만 담당 |
| **개방-폐쇄 (OCP)** | 확장에는 열려있고, 수정에는 닫혀있음 | Protocol과 Extension 활용 |
| **리스코프 치환 (LSP)** | 추상화를 구현체로 안전하게 대체 가능 | Protocol 준수 타입은 상호 교체 가능 |
| **인터페이스 분리 (ISP)** | 큰 범용 프로토콜 대신 집중된 작은 프로토콜 사용 | 목적별 Protocol 분리 |
| **의존성 역전 (DIP)** | 추상화에 의존하고, 구체 타입에 의존하지 않음 | @Environment, @EnvironmentObject로 주입 |

### SRP 예시
```swift
// 여러 책임을 가진 View - 피해야 함
struct BadUserView: View {
    @State private var user: User?

    var body: some View {
        // UI + 네트워크 호출 + 데이터 변환 모두 포함
    }

    func fetchUser() async { /* 네트워크 로직 */ }
    func formatDate(_ date: Date) -> String { /* 포맷팅 로직 */ }
}

// 단일 책임 분리 - 권장
struct GoodUserView: View {
    @StateObject private var viewModel = UserViewModel()

    var body: some View {
        // UI만 담당
    }
}

@MainActor
final class UserViewModel: ObservableObject {
    // 비즈니스 로직만 담당
}
```

### OCP 예시
```swift
// Protocol로 확장 가능한 구조
protocol DataFetchable {
    func fetch() async throws -> Data
}

// 기존 코드 수정 없이 새로운 구현 추가
struct APIFetcher: DataFetchable {
    func fetch() async throws -> Data { /* API 호출 */ }
}

struct CacheFetcher: DataFetchable {
    func fetch() async throws -> Data { /* 캐시 조회 */ }
}
```

### ISP 예시
```swift
// 큰 범용 프로토콜 - 피해야 함
protocol DataManager {
    func fetch() async throws -> Data
    func save(_ data: Data) async throws
    func delete() async throws
    func sync() async throws
}

// 작은 집중된 프로토콜 - 권장
protocol Fetchable {
    func fetch() async throws -> Data
}

protocol Savable {
    func save(_ data: Data) async throws
}

protocol Deletable {
    func delete() async throws
}
```

---

## 네이밍 컨벤션

| 항목 | 규칙 | 예시 |
|------|------|------|
| **타입** | UpperCamelCase | `UserProfileView`, `NetworkService` |
| **변수/상수** | lowerCamelCase | `userName`, `isLoading` |
| **Protocol** | 형용사 또는 ~able/~ible | `Configurable`, `DataFetchable` |
| **ViewModel** | *ViewModel 접미사 | `HomeViewModel`, `SettingsViewModel` |
| **View** | *View 접미사 | `ProfileView`, `ItemCardView` |
| **Service** | *Service 접미사 | `AuthService`, `NetworkService` |
| **Repository** | *Repository 접미사 | `UserRepository`, `CacheRepository` |
| **Boolean** | is/has/should 접두사 | `isLoading`, `hasError`, `shouldRefresh` |

---

## 주석 작성 규칙

### 기본 원칙
- 주석은 **한글(Korean)**로 작성하여 팀 가독성 향상
- 복잡한 로직이나 명확하지 않은 구현 세부사항에 주석 추가
- `// MARK:` 를 활용한 코드 섹션 구분

### 문서화 주석 형식
```swift
/// 사용자 데이터를 서버에서 가져옵니다.
/// - Parameter userId: 조회할 사용자의 고유 식별자
/// - Returns: 사용자 정보 객체
/// - Throws: 네트워크 오류 또는 파싱 오류
func fetchUser(userId: String) async throws -> User {
    // 구현
}
```

### MARK 주석 활용
```swift
// MARK: - Properties
// MARK: - Lifecycle
// MARK: - Public Methods
// MARK: - Private Methods
// MARK: - Actions
```

---

## 에러 처리

### 커스텀 에러 정의
```swift
/// 앱 전역에서 사용하는 에러 타입
enum AppError: LocalizedError {
    case networkFailure(underlying: Error)
    case invalidData
    case unauthorized

    var errorDescription: String? {
        switch self {
        case .networkFailure(let error):
            return "네트워크 오류: \(error.localizedDescription)"
        case .invalidData:
            return "데이터 형식이 올바르지 않습니다"
        case .unauthorized:
            return "인증이 필요합니다"
        }
    }
}
```

### 에러 처리 패턴
```swift
// Result 타입 활용
func fetchData() async -> Result<Data, AppError> {
    do {
        let data = try await networkService.fetch()
        return .success(data)
    } catch {
        return .failure(.networkFailure(underlying: error))
    }
}

// ViewModel에서의 에러 처리
@MainActor
final class ContentViewModel: ObservableObject {
    @Published private(set) var error: AppError?

    func loadData() async {
        do {
            // 데이터 로드
        } catch let error as AppError {
            self.error = error
        } catch {
            self.error = .networkFailure(underlying: error)
        }
    }
}
```
