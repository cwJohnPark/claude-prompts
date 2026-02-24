---
name: swiftui-architecture
description: SwiftUI MVVM 아키텍처 패턴, 프로젝트 구조, 의존성 주입 가이드
category: apple
tags: [swiftui, ios, macos, architecture, mvvm]
---

# SwiftUI 아키텍처 스킬

## 개요
SwiftUI + iOS/macOS 앱의 프로젝트 구조, MVVM 아키텍처 패턴, 의존성 주입 방법을 정의합니다.

---

## 프로젝트 구조 (관심사 분리)

```
ProjectName/
├── App/
│   └── ProjectNameApp.swift          # @main 진입점
│
├── Views/
│   ├── Components/                    # 재사용 가능한 UI 컴포넌트
│   │   ├── Buttons/
│   │   ├── Cards/
│   │   └── Inputs/
│   ├── Screens/                       # 화면 단위 View
│   │   ├── Home/
│   │   ├── Settings/
│   │   └── Detail/
│   └── Modifiers/                     # 커스텀 ViewModifier
│
├── ViewModels/
│   ├── HomeViewModel.swift
│   ├── SettingsViewModel.swift
│   └── DetailViewModel.swift
│
├── Models/
│   ├── Domain/                        # 도메인 엔티티 (비즈니스 모델)
│   │   ├── User.swift
│   │   └── Item.swift
│   └── DTO/                           # 데이터 전송 객체 (API 응답)
│       ├── UserDTO.swift
│       └── ItemDTO.swift
│
├── Services/
│   ├── Protocols/                     # 서비스 프로토콜 정의
│   │   ├── NetworkServiceProtocol.swift
│   │   └── AuthServiceProtocol.swift
│   └── Implementations/               # 서비스 구현체
│       ├── NetworkService.swift
│       └── AuthService.swift
│
├── Repositories/
│   ├── Protocols/                     # 저장소 프로토콜 정의
│   │   ├── UserRepositoryProtocol.swift
│   │   └── ItemRepositoryProtocol.swift
│   └── Implementations/               # 저장소 구현체
│       ├── UserRepository.swift
│       └── ItemRepository.swift
│
├── Utilities/
│   ├── Extensions/                    # Swift 타입 확장
│   │   ├── Date+Extensions.swift
│   │   └── String+Extensions.swift
│   └── Helpers/                       # 유틸리티 함수
│       └── DateFormatter+Helpers.swift
│
├── Resources/
│   ├── Assets.xcassets               # 이미지 및 색상 에셋
│   └── Localizable.strings           # 다국어 지원
│
└── Config/
    ├── AppConfiguration.swift         # 앱 설정 및 상수
    └── Environment.swift              # 환경 설정 (dev/prod)
```

---

## MVVM 아키텍처 패턴

### 계층별 책임

| 계층 | 책임 | 예시 |
|------|------|------|
| **View** | UI 렌더링, 사용자 입력 전달 | `HomeView.swift` |
| **ViewModel** | 비즈니스 로직, 상태 관리 | `HomeViewModel.swift` |
| **Model** | 데이터 구조 정의 | `User.swift`, `Item.swift` |
| **Service** | 외부 시스템 통신 | `NetworkService.swift` |
| **Repository** | 데이터 저장/조회 추상화 | `UserRepository.swift` |

### View 구현
```swift
import SwiftUI

/// 홈 화면 View
/// - 사용자 목록을 표시하고 상호작용을 처리합니다.
struct HomeView: View {
    // MARK: - Properties

    @StateObject private var viewModel: HomeViewModel

    // MARK: - Initialization

    init(viewModel: HomeViewModel = HomeViewModel()) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }

    // MARK: - Body

    var body: some View {
        NavigationStack {
            content
                .navigationTitle("홈")
                .toolbar { toolbarContent }
        }
        .task { await viewModel.loadData() }
        .alert("오류", isPresented: $viewModel.showError) {
            Button("확인", role: .cancel) { }
        } message: {
            Text(viewModel.errorMessage)
        }
    }

    // MARK: - Private Views

    @ViewBuilder
    private var content: some View {
        switch viewModel.state {
        case .idle, .loading:
            ProgressView("로딩 중...")
        case .loaded:
            userList
        case .error:
            errorView
        }
    }

    private var userList: some View {
        List(viewModel.users) { user in
            UserRow(user: user)
        }
    }

    private var errorView: some View {
        ContentUnavailableView(
            "데이터를 불러올 수 없습니다",
            systemImage: "exclamationmark.triangle",
            description: Text("다시 시도해 주세요")
        )
    }

    @ToolbarContentBuilder
    private var toolbarContent: some ToolbarContent {
        ToolbarItem(placement: .primaryAction) {
            Button("새로고침") {
                Task { await viewModel.refresh() }
            }
        }
    }
}
```

### ViewModel 구현
```swift
import Foundation
import Combine

/// 홈 화면의 비즈니스 로직을 담당하는 ViewModel
@MainActor
final class HomeViewModel: ObservableObject {

    // MARK: - Published Properties

    @Published private(set) var state: ViewState = .idle
    @Published private(set) var users: [User] = []
    @Published var showError = false
    @Published private(set) var errorMessage = ""

    // MARK: - Dependencies

    private let userRepository: UserRepositoryProtocol
    private var cancellables = Set<AnyCancellable>()

    // MARK: - View State

    enum ViewState {
        case idle
        case loading
        case loaded
        case error
    }

    // MARK: - Initialization

    init(userRepository: UserRepositoryProtocol = UserRepository()) {
        self.userRepository = userRepository
    }

    // MARK: - Public Methods

    /// 초기 데이터를 로드합니다.
    func loadData() async {
        guard state == .idle else { return }
        await fetchUsers()
    }

    /// 데이터를 새로고침합니다.
    func refresh() async {
        await fetchUsers()
    }

    // MARK: - Private Methods

    private func fetchUsers() async {
        state = .loading

        do {
            users = try await userRepository.fetchAll()
            state = .loaded
        } catch {
            state = .error
            errorMessage = error.localizedDescription
            showError = true
        }
    }
}
```

---

## 의존성 주입 패턴

### Protocol 기반 추상화
```swift
// MARK: - Protocol 정의

/// 사용자 저장소 프로토콜
protocol UserRepositoryProtocol {
    func fetchAll() async throws -> [User]
    func fetch(id: UUID) async throws -> User
    func save(_ user: User) async throws
    func delete(id: UUID) async throws
}

/// 네트워크 서비스 프로토콜
protocol NetworkServiceProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}
```

### 구현체
```swift
// MARK: - Repository 구현

final class UserRepository: UserRepositoryProtocol {

    private let networkService: NetworkServiceProtocol
    private let cacheService: CacheServiceProtocol

    init(
        networkService: NetworkServiceProtocol = NetworkService(),
        cacheService: CacheServiceProtocol = CacheService()
    ) {
        self.networkService = networkService
        self.cacheService = cacheService
    }

    func fetchAll() async throws -> [User] {
        // 캐시 확인
        if let cached: [User] = cacheService.get(key: "users") {
            return cached
        }

        // 네트워크 요청
        let dtos: [UserDTO] = try await networkService.request(.users)
        let users = dtos.map { $0.toDomain() }

        // 캐시 저장
        cacheService.set(key: "users", value: users)

        return users
    }

    // ... 기타 메서드
}
```

### Environment를 통한 주입
```swift
// MARK: - Environment Key 정의

private struct UserRepositoryKey: EnvironmentKey {
    static let defaultValue: UserRepositoryProtocol = UserRepository()
}

extension EnvironmentValues {
    var userRepository: UserRepositoryProtocol {
        get { self[UserRepositoryKey.self] }
        set { self[UserRepositoryKey.self] = newValue }
    }
}

// MARK: - 앱 진입점에서 주입

@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.userRepository, UserRepository())
        }
    }
}

// MARK: - View에서 사용

struct ContentView: View {
    @Environment(\.userRepository) private var userRepository

    var body: some View {
        // ...
    }
}
```

### EnvironmentObject를 통한 공유 상태
```swift
// MARK: - App State

@MainActor
final class AppState: ObservableObject {
    @Published var currentUser: User?
    @Published var isAuthenticated = false

    private let authService: AuthServiceProtocol

    init(authService: AuthServiceProtocol = AuthService()) {
        self.authService = authService
    }

    func signIn(email: String, password: String) async throws {
        currentUser = try await authService.signIn(email: email, password: password)
        isAuthenticated = true
    }

    func signOut() {
        currentUser = nil
        isAuthenticated = false
    }
}

// MARK: - 앱에서 주입

@main
struct MyApp: App {
    @StateObject private var appState = AppState()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}

// MARK: - View에서 사용

struct ProfileView: View {
    @EnvironmentObject private var appState: AppState

    var body: some View {
        if let user = appState.currentUser {
            Text("안녕하세요, \(user.name)님")
        }
    }
}
```

---

## Model 계층

### Domain Model (비즈니스 엔티티)
```swift
/// 사용자 도메인 모델
struct User: Identifiable, Equatable, Hashable {
    let id: UUID
    let name: String
    let email: String
    let createdAt: Date

    // 비즈니스 로직
    var displayName: String {
        name.isEmpty ? "익명 사용자" : name
    }
}
```

### DTO (Data Transfer Object)
```swift
/// API 응답용 사용자 DTO
struct UserDTO: Decodable {
    let id: String
    let name: String
    let email: String
    let createdAt: String

    /// Domain Model로 변환
    func toDomain() -> User {
        User(
            id: UUID(uuidString: id) ?? UUID(),
            name: name,
            email: email,
            createdAt: ISO8601DateFormatter().date(from: createdAt) ?? Date()
        )
    }
}
```

---

## Service 계층

### Network Service
```swift
/// 네트워크 요청을 처리하는 서비스
final class NetworkService: NetworkServiceProtocol {

    private let session: URLSession
    private let decoder: JSONDecoder

    init(session: URLSession = .shared) {
        self.session = session
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
    }

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let request = try endpoint.urlRequest()
        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(statusCode: httpResponse.statusCode)
        }

        return try decoder.decode(T.self, from: data)
    }
}

// MARK: - Endpoint 정의

enum Endpoint {
    case users
    case user(id: UUID)

    func urlRequest() throws -> URLRequest {
        var components = URLComponents()
        components.scheme = "https"
        components.host = AppConfiguration.apiHost
        components.path = path

        guard let url = components.url else {
            throw NetworkError.invalidURL
        }

        var request = URLRequest(url: url)
        request.httpMethod = method
        return request
    }

    private var path: String {
        switch self {
        case .users: return "/api/users"
        case .user(let id): return "/api/users/\(id)"
        }
    }

    private var method: String {
        switch self {
        case .users, .user: return "GET"
        }
    }
}
```

---

## 성능 최적화

### SwiftUI 렌더링 최적화
```swift
// Equatable 구현으로 불필요한 재렌더링 방지
struct UserRow: View, Equatable {
    let user: User

    static func == (lhs: UserRow, rhs: UserRow) -> Bool {
        lhs.user.id == rhs.user.id
    }

    var body: some View {
        HStack {
            Text(user.name)
            Spacer()
            Text(user.email)
        }
    }
}

// 큰 리스트는 Lazy 컴포넌트 사용
struct UserListView: View {
    let users: [User]

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 8) {
                ForEach(users) { user in
                    UserRow(user: user)
                }
            }
        }
    }
}
```

### 메모리 관리
```swift
// 클로저에서 weak self 사용
final class SomeViewModel: ObservableObject {
    private var cancellables = Set<AnyCancellable>()

    func observe() {
        somePublisher
            .sink { [weak self] value in
                self?.handleValue(value)
            }
            .store(in: &cancellables)
    }
}
```

---

## 참고 자료

- [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui/)
- [Combine Framework](https://developer.apple.com/documentation/combine)
