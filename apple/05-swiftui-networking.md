---
name: swiftui-networking
description: iOS 앱 네트워킹 완전 가이드 (URLSession, async/await, REST API, 인증, 캐싱)
category: apple
tags: [swiftui, ios, networking, urlsession, rest-api]
---

# iOS 네트워킹 스킬

## 개요

iOS 앱의 네트워크 통신을 위한 완전 가이드입니다. URLSession, async/await, REST API, JSON 처리, 에러 핸들링, 인증, 캐싱을 다룹니다.

---

## 아키텍처 개요

```
┌─────────────────────────────────────────────────────┐
│                    View / ViewModel                  │
└─────────────────────────┬───────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────┐
│                   APIClient                          │
│  - 엔드포인트 정의                                    │
│  - 요청 생성                                         │
│  - 응답 처리                                         │
└─────────────────────────┬───────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────┐
│               NetworkService                         │
│  - URLSession 래퍼                                   │
│  - 인증 처리                                         │
│  - 에러 변환                                         │
│  - 재시도 로직                                       │
└─────────────────────────┬───────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────┐
│                  URLSession                          │
└─────────────────────────────────────────────────────┘
```

---

## 기본 설정

### Endpoint 정의

```swift
import Foundation

// MARK: - HTTP Method

enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}

// MARK: - Endpoint Protocol

protocol Endpoint {
    var baseURL: URL { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var headers: [String: String]? { get }
    var queryItems: [URLQueryItem]? { get }
    var body: Data? { get }
}

extension Endpoint {
    var baseURL: URL {
        URL(string: "https://api.example.com")!
    }

    var headers: [String: String]? {
        ["Content-Type": "application/json"]
    }

    var queryItems: [URLQueryItem]? { nil }
    var body: Data? { nil }

    var url: URL {
        var components = URLComponents(url: baseURL.appendingPathComponent(path), resolvingAgainstBaseURL: true)!
        components.queryItems = queryItems
        return components.url!
    }

    var urlRequest: URLRequest {
        var request = URLRequest(url: url)
        request.httpMethod = method.rawValue
        request.httpBody = body
        headers?.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        return request
    }
}
```

### API Endpoints 구현

```swift
// MARK: - User API Endpoints

enum UserEndpoint: Endpoint {
    case getUser(id: String)
    case getUsers(page: Int, limit: Int)
    case createUser(CreateUserRequest)
    case updateUser(id: String, UpdateUserRequest)
    case deleteUser(id: String)
    case uploadAvatar(userId: String, imageData: Data)

    var path: String {
        switch self {
        case .getUser(let id):
            return "/users/\(id)"
        case .getUsers:
            return "/users"
        case .createUser:
            return "/users"
        case .updateUser(let id, _):
            return "/users/\(id)"
        case .deleteUser(let id):
            return "/users/\(id)"
        case .uploadAvatar(let userId, _):
            return "/users/\(userId)/avatar"
        }
    }

    var method: HTTPMethod {
        switch self {
        case .getUser, .getUsers:
            return .get
        case .createUser, .uploadAvatar:
            return .post
        case .updateUser:
            return .put
        case .deleteUser:
            return .delete
        }
    }

    var queryItems: [URLQueryItem]? {
        switch self {
        case .getUsers(let page, let limit):
            return [
                URLQueryItem(name: "page", value: "\(page)"),
                URLQueryItem(name: "limit", value: "\(limit)")
            ]
        default:
            return nil
        }
    }

    var body: Data? {
        switch self {
        case .createUser(let request):
            return try? JSONEncoder().encode(request)
        case .updateUser(_, let request):
            return try? JSONEncoder().encode(request)
        case .uploadAvatar(_, let imageData):
            return imageData
        default:
            return nil
        }
    }

    var headers: [String: String]? {
        switch self {
        case .uploadAvatar:
            return ["Content-Type": "image/jpeg"]
        default:
            return ["Content-Type": "application/json"]
        }
    }
}

// MARK: - Request/Response Models

struct CreateUserRequest: Encodable {
    let name: String
    let email: String
    let password: String
}

struct UpdateUserRequest: Encodable {
    let name: String?
    let email: String?
}
```

---

## NetworkService 구현

### Network Error 정의

```swift
import Foundation

// MARK: - Network Error

enum NetworkError: LocalizedError, Equatable {
    case invalidURL
    case invalidResponse
    case invalidData
    case decodingError(String)
    case serverError(statusCode: Int, message: String?)
    case unauthorized
    case forbidden
    case notFound
    case noInternetConnection
    case timeout
    case unknown(String)

    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "잘못된 URL입니다"
        case .invalidResponse:
            return "서버로부터 잘못된 응답을 받았습니다"
        case .invalidData:
            return "잘못된 데이터를 받았습니다"
        case .decodingError(let message):
            return "디코딩 오류: \(message)"
        case .serverError(let statusCode, let message):
            return "서버 오류 (\(statusCode)): \(message ?? "알 수 없음")"
        case .unauthorized:
            return "인증이 필요합니다. 다시 로그인해주세요."
        case .forbidden:
            return "접근 권한이 없습니다"
        case .notFound:
            return "리소스를 찾을 수 없습니다"
        case .noInternetConnection:
            return "인터넷 연결이 없습니다"
        case .timeout:
            return "요청 시간이 초과되었습니다"
        case .unknown(let message):
            return message
        }
    }

    // Equatable 준수를 위한 구현
    static func == (lhs: NetworkError, rhs: NetworkError) -> Bool {
        switch (lhs, rhs) {
        case (.invalidURL, .invalidURL),
             (.invalidResponse, .invalidResponse),
             (.invalidData, .invalidData),
             (.unauthorized, .unauthorized),
             (.forbidden, .forbidden),
             (.notFound, .notFound),
             (.noInternetConnection, .noInternetConnection),
             (.timeout, .timeout):
            return true
        case (.decodingError(let lMsg), .decodingError(let rMsg)):
            return lMsg == rMsg
        case (.serverError(let lCode, let lMsg), .serverError(let rCode, let rMsg)):
            return lCode == rCode && lMsg == rMsg
        case (.unknown(let lMsg), .unknown(let rMsg)):
            return lMsg == rMsg
        default:
            return false
        }
    }
}
```

### Network Service Protocol

```swift
// MARK: - Network Service Protocol

protocol NetworkServiceProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
    func request(_ endpoint: Endpoint) async throws -> Data
    func request(_ endpoint: Endpoint) async throws
}
```

### Network Service 구현

```swift
// MARK: - Network Service

final class NetworkService: NetworkServiceProtocol {

    static let shared = NetworkService()

    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder

    init(session: URLSession = .shared) {
        self.session = session

        self.decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        decoder.dateDecodingStrategy = .iso8601

        self.encoder = JSONEncoder()
        encoder.keyEncodingStrategy = .convertToSnakeCase
        encoder.dateEncodingStrategy = .iso8601
    }

    // MARK: - Decodable 응답

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let data = try await request(endpoint)

        do {
            return try decoder.decode(T.self, from: data)
        } catch {
            throw NetworkError.decodingError(error.localizedDescription)
        }
    }

    // MARK: - Data 응답

    func request(_ endpoint: Endpoint) async throws -> Data {
        let request = endpoint.urlRequest

        #if DEBUG
        logRequest(request)
        #endif

        let (data, response): (Data, URLResponse)

        do {
            (data, response) = try await session.data(for: request)
        } catch let error as URLError {
            throw mapURLError(error)
        } catch {
            throw NetworkError.unknown(error.localizedDescription)
        }

        #if DEBUG
        logResponse(response, data: data)
        #endif

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        try validateResponse(httpResponse, data: data)

        return data
    }

    // MARK: - Void 응답 (204 No Content 등)

    func request(_ endpoint: Endpoint) async throws {
        let _: Data = try await request(endpoint)
    }

    // MARK: - Private Methods

    private func validateResponse(_ response: HTTPURLResponse, data: Data) throws {
        switch response.statusCode {
        case 200...299:
            return
        case 401:
            throw NetworkError.unauthorized
        case 403:
            throw NetworkError.forbidden
        case 404:
            throw NetworkError.notFound
        case 400...499:
            let message = extractErrorMessage(from: data)
            throw NetworkError.serverError(statusCode: response.statusCode, message: message)
        case 500...599:
            let message = extractErrorMessage(from: data)
            throw NetworkError.serverError(statusCode: response.statusCode, message: message)
        default:
            throw NetworkError.invalidResponse
        }
    }

    private func extractErrorMessage(from data: Data) -> String? {
        struct ErrorResponse: Decodable {
            let message: String?
            let error: String?
        }

        if let errorResponse = try? decoder.decode(ErrorResponse.self, from: data) {
            return errorResponse.message ?? errorResponse.error
        }

        return String(data: data, encoding: .utf8)
    }

    private func mapURLError(_ error: URLError) -> NetworkError {
        switch error.code {
        case .notConnectedToInternet, .networkConnectionLost:
            return .noInternetConnection
        case .timedOut:
            return .timeout
        default:
            return .unknown(error.localizedDescription)
        }
    }

    // MARK: - Logging

    private func logRequest(_ request: URLRequest) {
        print("🌐 REQUEST: \(request.httpMethod ?? "?") \(request.url?.absoluteString ?? "?")")
        if let headers = request.allHTTPHeaderFields {
            print("📋 Headers: \(headers)")
        }
        if let body = request.httpBody, let bodyString = String(data: body, encoding: .utf8) {
            print("📦 Body: \(bodyString)")
        }
    }

    private func logResponse(_ response: URLResponse, data: Data) {
        if let httpResponse = response as? HTTPURLResponse {
            print("✅ RESPONSE: \(httpResponse.statusCode)")
        }
        if let json = try? JSONSerialization.jsonObject(with: data),
           let prettyData = try? JSONSerialization.data(withJSONObject: json, options: .prettyPrinted),
           let prettyString = String(data: prettyData, encoding: .utf8) {
            print("📥 Data: \(prettyString)")
        }
    }
}
```

---

## APIClient 구현

### 도메인별 API Client

```swift
// MARK: - User API Client Protocol

protocol UserAPIClientProtocol {
    func getUser(id: String) async throws -> User
    func getUsers(page: Int, limit: Int) async throws -> PaginatedResponse<User>
    func createUser(_ request: CreateUserRequest) async throws -> User
    func updateUser(id: String, _ request: UpdateUserRequest) async throws -> User
    func deleteUser(id: String) async throws
}

// MARK: - User API Client

final class UserAPIClient: UserAPIClientProtocol {

    private let networkService: NetworkServiceProtocol

    init(networkService: NetworkServiceProtocol = NetworkService.shared) {
        self.networkService = networkService
    }

    func getUser(id: String) async throws -> User {
        try await networkService.request(UserEndpoint.getUser(id: id))
    }

    func getUsers(page: Int = 1, limit: Int = 20) async throws -> PaginatedResponse<User> {
        try await networkService.request(UserEndpoint.getUsers(page: page, limit: limit))
    }

    func createUser(_ request: CreateUserRequest) async throws -> User {
        try await networkService.request(UserEndpoint.createUser(request))
    }

    func updateUser(id: String, _ request: UpdateUserRequest) async throws -> User {
        try await networkService.request(UserEndpoint.updateUser(id: id, request))
    }

    func deleteUser(id: String) async throws {
        try await networkService.request(UserEndpoint.deleteUser(id: id))
    }
}

// MARK: - Response Models

struct User: Codable, Identifiable, Equatable {
    let id: String
    let name: String
    let email: String
    let avatarUrl: String?
    let createdAt: Date?
}

struct PaginatedResponse<T: Codable>: Codable {
    let data: [T]
    let page: Int
    let totalPages: Int
    let totalCount: Int

    var hasNextPage: Bool {
        page < totalPages
    }
}
```

---

## 인증 처리

### Token Manager

```swift
import Foundation

// MARK: - Auth Token

struct AuthToken: Codable {
    let accessToken: String
    let refreshToken: String
    let expiresAt: Date

    var isExpired: Bool {
        Date() >= expiresAt
    }

    var isExpiringSoon: Bool {
        Date().addingTimeInterval(60) >= expiresAt  // 1분 이내 만료
    }
}

// MARK: - Token Manager

actor TokenManager {

    static let shared = TokenManager()

    private let keychain = KeychainService.shared
    private let tokenKey = "auth_token"

    private var currentToken: AuthToken?
    private var refreshTask: Task<AuthToken, Error>?

    private init() {
        // 저장된 토큰 로드
        currentToken = try? keychain.load(AuthToken.self, forKey: tokenKey)
    }

    // MARK: - Public Methods

    func getValidToken() async throws -> String {
        // 토큰 없음
        guard let token = currentToken else {
            throw NetworkError.unauthorized
        }

        // 토큰 유효
        if !token.isExpired && !token.isExpiringSoon {
            return token.accessToken
        }

        // 이미 갱신 중이면 대기
        if let refreshTask = refreshTask {
            return try await refreshTask.value.accessToken
        }

        // 토큰 갱신
        let task = Task { () -> AuthToken in
            defer { refreshTask = nil }
            return try await refreshToken(token.refreshToken)
        }

        refreshTask = task
        return try await task.value.accessToken
    }

    func setToken(_ token: AuthToken) throws {
        currentToken = token
        try keychain.save(token, forKey: tokenKey)
    }

    func clearToken() throws {
        currentToken = nil
        try keychain.delete(forKey: tokenKey)
    }

    // MARK: - Private Methods

    private func refreshToken(_ refreshToken: String) async throws -> AuthToken {
        // 토큰 갱신 API 호출
        let endpoint = AuthEndpoint.refreshToken(refreshToken: refreshToken)
        let newToken: AuthToken = try await NetworkService.shared.request(endpoint)

        try setToken(newToken)
        return newToken
    }
}
```

### 인증 포함 NetworkService

```swift
// MARK: - Authenticated Network Service

final class AuthenticatedNetworkService: NetworkServiceProtocol {

    static let shared = AuthenticatedNetworkService()

    private let baseService: NetworkService
    private let tokenManager: TokenManager

    init(
        baseService: NetworkService = .shared,
        tokenManager: TokenManager = .shared
    ) {
        self.baseService = baseService
        self.tokenManager = tokenManager
    }

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let authenticatedEndpoint = try await addAuthHeader(to: endpoint)

        do {
            return try await baseService.request(authenticatedEndpoint)
        } catch NetworkError.unauthorized {
            // 토큰 만료 시 갱신 후 재시도
            try await tokenManager.clearToken()
            throw NetworkError.unauthorized
        }
    }

    func request(_ endpoint: Endpoint) async throws -> Data {
        let authenticatedEndpoint = try await addAuthHeader(to: endpoint)
        return try await baseService.request(authenticatedEndpoint)
    }

    func request(_ endpoint: Endpoint) async throws {
        let authenticatedEndpoint = try await addAuthHeader(to: endpoint)
        try await baseService.request(authenticatedEndpoint)
    }

    private func addAuthHeader(to endpoint: Endpoint) async throws -> AuthenticatedEndpoint {
        let token = try await tokenManager.getValidToken()
        return AuthenticatedEndpoint(base: endpoint, token: token)
    }
}

// MARK: - Authenticated Endpoint Wrapper

struct AuthenticatedEndpoint: Endpoint {
    let base: Endpoint
    let token: String

    var baseURL: URL { base.baseURL }
    var path: String { base.path }
    var method: HTTPMethod { base.method }
    var queryItems: [URLQueryItem]? { base.queryItems }
    var body: Data? { base.body }

    var headers: [String: String]? {
        var headers = base.headers ?? [:]
        headers["Authorization"] = "Bearer \(token)"
        return headers
    }
}
```

---

## 재시도 로직

### Retry Policy

```swift
// MARK: - Retry Policy

struct RetryPolicy {
    let maxRetries: Int
    let retryableStatusCodes: Set<Int>
    let delay: (Int) -> TimeInterval  // 재시도 횟수에 따른 지연 시간

    static let `default` = RetryPolicy(
        maxRetries: 3,
        retryableStatusCodes: [408, 429, 500, 502, 503, 504],
        delay: { attempt in
            // Exponential backoff: 1초, 2초, 4초...
            TimeInterval(pow(2.0, Double(attempt - 1)))
        }
    )

    static let aggressive = RetryPolicy(
        maxRetries: 5,
        retryableStatusCodes: [408, 429, 500, 502, 503, 504],
        delay: { attempt in
            // 더 짧은 지연
            TimeInterval(attempt) * 0.5
        }
    )

    static let none = RetryPolicy(
        maxRetries: 0,
        retryableStatusCodes: [],
        delay: { _ in 0 }
    )
}
```

### Retrying Network Service

```swift
// MARK: - Retrying Network Service

final class RetryingNetworkService: NetworkServiceProtocol {

    private let baseService: NetworkServiceProtocol
    private let policy: RetryPolicy

    init(
        baseService: NetworkServiceProtocol = NetworkService.shared,
        policy: RetryPolicy = .default
    ) {
        self.baseService = baseService
        self.policy = policy
    }

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        try await withRetry { [baseService] in
            try await baseService.request(endpoint)
        }
    }

    func request(_ endpoint: Endpoint) async throws -> Data {
        try await withRetry { [baseService] in
            try await baseService.request(endpoint)
        }
    }

    func request(_ endpoint: Endpoint) async throws {
        try await withRetry { [baseService] in
            try await baseService.request(endpoint)
        }
    }

    private func withRetry<T>(_ operation: @escaping () async throws -> T) async throws -> T {
        var lastError: Error?

        for attempt in 0...policy.maxRetries {
            do {
                return try await operation()
            } catch let error as NetworkError {
                lastError = error

                guard shouldRetry(error: error, attempt: attempt) else {
                    throw error
                }

                let delay = policy.delay(attempt + 1)
                print("⏳ 재시도 \(attempt + 1)회 (\(delay)초 후)")
                try await Task.sleep(nanoseconds: UInt64(delay * 1_000_000_000))

            } catch {
                throw error
            }
        }

        throw lastError ?? NetworkError.unknown("재시도 실패")
    }

    private func shouldRetry(error: NetworkError, attempt: Int) -> Bool {
        guard attempt < policy.maxRetries else { return false }

        switch error {
        case .noInternetConnection, .timeout:
            return true
        case .serverError(let statusCode, _):
            return policy.retryableStatusCodes.contains(statusCode)
        default:
            return false
        }
    }
}
```

---

## 네트워크 모니터링

### Network Monitor

```swift
import Network
import Combine

// MARK: - Network Monitor

final class NetworkMonitor: ObservableObject {

    static let shared = NetworkMonitor()

    @Published private(set) var isConnected = true
    @Published private(set) var connectionType: ConnectionType = .unknown

    private let monitor: NWPathMonitor
    private let queue = DispatchQueue(label: "NetworkMonitor")

    enum ConnectionType {
        case wifi
        case cellular
        case ethernet
        case unknown
    }

    private init() {
        monitor = NWPathMonitor()
        startMonitoring()
    }

    deinit {
        stopMonitoring()
    }

    func startMonitoring() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.connectionType = self?.getConnectionType(path) ?? .unknown
            }
        }
        monitor.start(queue: queue)
    }

    func stopMonitoring() {
        monitor.cancel()
    }

    private func getConnectionType(_ path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) {
            return .wifi
        } else if path.usesInterfaceType(.cellular) {
            return .cellular
        } else if path.usesInterfaceType(.wiredEthernet) {
            return .ethernet
        }
        return .unknown
    }
}
```

### SwiftUI에서 네트워크 상태 표시

```swift
// MARK: - SwiftUI에서 사용

struct ContentView: View {
    @StateObject private var networkMonitor = NetworkMonitor.shared

    var body: some View {
        VStack {
            if !networkMonitor.isConnected {
                NoInternetBanner()
            }

            // 메인 콘텐츠
        }
    }
}

struct NoInternetBanner: View {
    var body: some View {
        HStack {
            Image(systemName: "wifi.slash")
            Text("인터넷 연결이 없습니다")
        }
        .padding()
        .background(Color.red.opacity(0.8))
        .foregroundColor(.white)
    }
}
```

---

## 캐싱

### URL Cache 설정

```swift
// MARK: - Cache Configuration

extension URLSession {

    static var cached: URLSession {
        let config = URLSessionConfiguration.default

        // 메모리 캐시: 50MB, 디스크 캐시: 100MB
        let cache = URLCache(
            memoryCapacity: 50 * 1024 * 1024,
            diskCapacity: 100 * 1024 * 1024,
            diskPath: "network_cache"
        )

        config.urlCache = cache
        config.requestCachePolicy = .returnCacheDataElseLoad

        return URLSession(configuration: config)
    }
}
```

### Cache Policy

```swift
// MARK: - Cache Policy

enum CachePolicy {
    case networkOnly           // 항상 네트워크
    case cacheFirst           // 캐시 우선, 없으면 네트워크
    case cacheFirstThenNetwork // 캐시 먼저 반환, 백그라운드에서 네트워크 갱신
    case networkFirstThenCache // 네트워크 우선, 실패 시 캐시
}

// MARK: - Cacheable Response

protocol CacheableResponse {
    static var cacheKey: String { get }
    static var cacheDuration: TimeInterval { get }
}

extension User: CacheableResponse {
    static var cacheKey: String { "user" }
    static var cacheDuration: TimeInterval { 60 * 5 }  // 5분
}
```

### Response Cache

```swift
import CommonCrypto

// MARK: - Response Cache

actor ResponseCache {

    static let shared = ResponseCache()

    private var storage: [String: CacheEntry] = [:]
    private let fileManager = FileManager.default
    private let cacheDirectory: URL

    private struct CacheEntry {
        let data: Data
        let timestamp: Date
        let duration: TimeInterval

        var isExpired: Bool {
            Date().timeIntervalSince(timestamp) > duration
        }
    }

    private init() {
        let paths = fileManager.urls(for: .cachesDirectory, in: .userDomainMask)
        cacheDirectory = paths[0].appendingPathComponent("ResponseCache")

        try? fileManager.createDirectory(at: cacheDirectory, withIntermediateDirectories: true)
    }

    // MARK: - Memory Cache

    func get<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        guard let entry = storage[key], !entry.isExpired else {
            storage.removeValue(forKey: key)
            return nil
        }

        return try? JSONDecoder().decode(T.self, from: entry.data)
    }

    func set<T: Encodable>(_ value: T, forKey key: String, duration: TimeInterval) {
        guard let data = try? JSONEncoder().encode(value) else { return }

        storage[key] = CacheEntry(
            data: data,
            timestamp: Date(),
            duration: duration
        )
    }

    func remove(forKey key: String) {
        storage.removeValue(forKey: key)
    }

    func clearAll() {
        storage.removeAll()
    }

    // MARK: - Disk Cache

    func saveToDisk<T: Encodable>(_ value: T, forKey key: String) {
        guard let data = try? JSONEncoder().encode(value) else { return }

        let fileURL = cacheDirectory.appendingPathComponent(key.sha256Hash)
        try? data.write(to: fileURL)
    }

    func loadFromDisk<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        let fileURL = cacheDirectory.appendingPathComponent(key.sha256Hash)

        guard let data = try? Data(contentsOf: fileURL) else { return nil }
        return try? JSONDecoder().decode(T.self, from: data)
    }
}

// MARK: - String Hash Extension

extension String {
    var sha256Hash: String {
        let data = Data(utf8)
        var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        data.withUnsafeBytes {
            _ = CC_SHA256($0.baseAddress, CC_LONG(data.count), &hash)
        }
        return hash.map { String(format: "%02x", $0) }.joined()
    }
}
```

### Caching Network Service

```swift
// MARK: - Caching Network Service

final class CachingNetworkService {

    private let networkService: NetworkServiceProtocol
    private let cache: ResponseCache

    init(
        networkService: NetworkServiceProtocol = NetworkService.shared,
        cache: ResponseCache = .shared
    ) {
        self.networkService = networkService
        self.cache = cache
    }

    func request<T: Decodable & CacheableResponse>(
        _ endpoint: Endpoint,
        cachePolicy: CachePolicy = .cacheFirstThenNetwork
    ) async throws -> T {
        let cacheKey = "\(T.cacheKey)_\(endpoint.url.absoluteString)"

        switch cachePolicy {
        case .networkOnly:
            let result: T = try await networkService.request(endpoint)
            await cache.set(result, forKey: cacheKey, duration: T.cacheDuration)
            return result

        case .cacheFirst:
            if let cached: T = await cache.get(T.self, forKey: cacheKey) {
                return cached
            }
            let result: T = try await networkService.request(endpoint)
            await cache.set(result, forKey: cacheKey, duration: T.cacheDuration)
            return result

        case .cacheFirstThenNetwork:
            // 캐시 반환 후 백그라운드 갱신
            if let cached: T = await cache.get(T.self, forKey: cacheKey) {
                Task {
                    if let fresh: T = try? await networkService.request(endpoint) {
                        await cache.set(fresh, forKey: cacheKey, duration: T.cacheDuration)
                    }
                }
                return cached
            }
            let result: T = try await networkService.request(endpoint)
            await cache.set(result, forKey: cacheKey, duration: T.cacheDuration)
            return result

        case .networkFirstThenCache:
            do {
                let result: T = try await networkService.request(endpoint)
                await cache.set(result, forKey: cacheKey, duration: T.cacheDuration)
                return result
            } catch {
                if let cached: T = await cache.get(T.self, forKey: cacheKey) {
                    return cached
                }
                throw error
            }
        }
    }
}
```

---

## 파일 업로드/다운로드

### Multipart Form Data

```swift
// MARK: - Multipart Form Data

struct MultipartFormData {

    private let boundary: String
    private var data: Data

    init() {
        boundary = "Boundary-\(UUID().uuidString)"
        data = Data()
    }

    var contentType: String {
        "multipart/form-data; boundary=\(boundary)"
    }

    mutating func append(_ value: String, withName name: String) {
        data.append("--\(boundary)\r\n".data(using: .utf8)!)
        data.append("Content-Disposition: form-data; name=\"\(name)\"\r\n\r\n".data(using: .utf8)!)
        data.append("\(value)\r\n".data(using: .utf8)!)
    }

    mutating func append(
        _ fileData: Data,
        withName name: String,
        fileName: String,
        mimeType: String
    ) {
        data.append("--\(boundary)\r\n".data(using: .utf8)!)
        data.append("Content-Disposition: form-data; name=\"\(name)\"; filename=\"\(fileName)\"\r\n".data(using: .utf8)!)
        data.append("Content-Type: \(mimeType)\r\n\r\n".data(using: .utf8)!)
        data.append(fileData)
        data.append("\r\n".data(using: .utf8)!)
    }

    mutating func finalize() -> Data {
        data.append("--\(boundary)--\r\n".data(using: .utf8)!)
        return data
    }
}
```

### Upload Service

```swift
// MARK: - Upload Service

final class UploadService {

    private let session: URLSession

    init(session: URLSession = .shared) {
        self.session = session
    }

    func uploadImage(
        _ imageData: Data,
        to url: URL,
        fileName: String = "image.jpg",
        additionalFields: [String: String] = [:]
    ) async throws -> Data {

        var formData = MultipartFormData()

        // 추가 필드
        for (key, value) in additionalFields {
            formData.append(value, withName: key)
        }

        // 이미지 파일
        formData.append(
            imageData,
            withName: "file",
            fileName: fileName,
            mimeType: "image/jpeg"
        )

        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue(formData.contentType, forHTTPHeaderField: "Content-Type")
        request.httpBody = formData.finalize()

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }

        return data
    }
}
```

### Download Service (Progress 포함)

```swift
// MARK: - Download Service

final class DownloadService: NSObject {

    private var session: URLSession!
    private var progressHandlers: [URLSessionTask: (Double) -> Void] = [:]
    private var completionHandlers: [URLSessionTask: (Result<URL, Error>) -> Void] = [:]

    override init() {
        super.init()
        let config = URLSessionConfiguration.default
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }

    func download(
        from url: URL,
        progress: @escaping (Double) -> Void,
        completion: @escaping (Result<URL, Error>) -> Void
    ) -> URLSessionDownloadTask {
        let task = session.downloadTask(with: url)
        progressHandlers[task] = progress
        completionHandlers[task] = completion
        task.resume()
        return task
    }

    // async/await 버전
    func download(from url: URL, progress: @escaping (Double) -> Void) async throws -> URL {
        try await withCheckedThrowingContinuation { continuation in
            _ = download(from: url, progress: progress) { result in
                continuation.resume(with: result)
            }
        }
    }
}

extension DownloadService: URLSessionDownloadDelegate {

    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didFinishDownloadingTo location: URL
    ) {
        // 임시 파일을 영구 위치로 이동
        let documentsPath = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        let destinationURL = documentsPath.appendingPathComponent(
            downloadTask.response?.suggestedFilename ?? "download"
        )

        try? FileManager.default.removeItem(at: destinationURL)

        do {
            try FileManager.default.moveItem(at: location, to: destinationURL)
            completionHandlers[downloadTask]?(.success(destinationURL))
        } catch {
            completionHandlers[downloadTask]?(.failure(error))
        }

        progressHandlers.removeValue(forKey: downloadTask)
        completionHandlers.removeValue(forKey: downloadTask)
    }

    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didWriteData bytesWritten: Int64,
        totalBytesWritten: Int64,
        totalBytesExpectedToWrite: Int64
    ) {
        let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)

        DispatchQueue.main.async { [weak self] in
            self?.progressHandlers[downloadTask]?(progress)
        }
    }

    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didCompleteWithError error: Error?
    ) {
        if let error = error {
            completionHandlers[task]?(.failure(error))
        }

        progressHandlers.removeValue(forKey: task)
        completionHandlers.removeValue(forKey: task)
    }
}
```

---

## ViewModel에서 사용

### 기본 사용 패턴

```swift
import SwiftUI
import Combine

@MainActor
final class UserListViewModel: ObservableObject {

    // MARK: - Published Properties

    @Published private(set) var users: [User] = []
    @Published private(set) var isLoading = false
    @Published private(set) var error: NetworkError?
    @Published private(set) var hasMorePages = true

    // MARK: - Private Properties

    private let apiClient: UserAPIClientProtocol
    private var currentPage = 1
    private let pageSize = 20

    // MARK: - Init

    init(apiClient: UserAPIClientProtocol = UserAPIClient()) {
        self.apiClient = apiClient
    }

    // MARK: - Public Methods

    func loadUsers() async {
        guard !isLoading else { return }

        isLoading = true
        error = nil
        currentPage = 1

        do {
            let response = try await apiClient.getUsers(page: currentPage, limit: pageSize)
            users = response.data
            hasMorePages = response.hasNextPage
        } catch let networkError as NetworkError {
            error = networkError
        } catch {
            self.error = .unknown(error.localizedDescription)
        }

        isLoading = false
    }

    func loadMoreUsers() async {
        guard !isLoading, hasMorePages else { return }

        isLoading = true
        currentPage += 1

        do {
            let response = try await apiClient.getUsers(page: currentPage, limit: pageSize)
            users.append(contentsOf: response.data)
            hasMorePages = response.hasNextPage
        } catch {
            currentPage -= 1  // 롤백
        }

        isLoading = false
    }

    func refresh() async {
        await loadUsers()
    }

    func deleteUser(_ user: User) async {
        do {
            try await apiClient.deleteUser(id: user.id)
            users.removeAll { $0.id == user.id }
        } catch let networkError as NetworkError {
            error = networkError
        } catch {
            self.error = .unknown(error.localizedDescription)
        }
    }
}
```

### SwiftUI View

```swift
struct UserListView: View {

    @StateObject private var viewModel = UserListViewModel()

    var body: some View {
        NavigationStack {
            Group {
                if viewModel.isLoading && viewModel.users.isEmpty {
                    ProgressView()
                } else if let error = viewModel.error, viewModel.users.isEmpty {
                    ErrorView(error: error) {
                        Task { await viewModel.loadUsers() }
                    }
                } else {
                    userList
                }
            }
            .navigationTitle("사용자")
            .refreshable {
                await viewModel.refresh()
            }
            .task {
                await viewModel.loadUsers()
            }
        }
    }

    private var userList: some View {
        List {
            ForEach(viewModel.users) { user in
                UserRow(user: user)
            }
            .onDelete { indexSet in
                Task {
                    for index in indexSet {
                        await viewModel.deleteUser(viewModel.users[index])
                    }
                }
            }

            // 무한 스크롤
            if viewModel.hasMorePages {
                ProgressView()
                    .frame(maxWidth: .infinity)
                    .task {
                        await viewModel.loadMoreUsers()
                    }
            }
        }
    }
}

struct UserRow: View {
    let user: User

    var body: some View {
        VStack(alignment: .leading) {
            Text(user.name)
                .font(.headline)
            Text(user.email)
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
    }
}

struct ErrorView: View {
    let error: NetworkError
    let retryAction: () -> Void

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
                .font(.largeTitle)
                .foregroundColor(.red)

            Text(error.localizedDescription)
                .multilineTextAlignment(.center)

            Button("다시 시도", action: retryAction)
                .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## 테스트

### Mock Network Service

```swift
// MARK: - Mock Network Service

final class MockNetworkService: NetworkServiceProtocol {

    var requestHandler: ((Endpoint) async throws -> Data)?
    var requestCallCount = 0
    var lastEndpoint: Endpoint?

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let data = try await request(endpoint)
        return try JSONDecoder().decode(T.self, from: data)
    }

    func request(_ endpoint: Endpoint) async throws -> Data {
        requestCallCount += 1
        lastEndpoint = endpoint

        guard let handler = requestHandler else {
            throw NetworkError.invalidResponse
        }

        return try await handler(endpoint)
    }

    func request(_ endpoint: Endpoint) async throws {
        _ = try await request(endpoint) as Data
    }

    // MARK: - Helpers

    func setResponse<T: Encodable>(_ response: T) {
        requestHandler = { _ in
            try JSONEncoder().encode(response)
        }
    }

    func setError(_ error: NetworkError) {
        requestHandler = { _ in
            throw error
        }
    }

    func reset() {
        requestHandler = nil
        requestCallCount = 0
        lastEndpoint = nil
    }
}
```

### URLProtocol Mock

```swift
// MARK: - Mock URL Protocol

final class MockURLProtocol: URLProtocol {

    static var requestHandler: ((URLRequest) throws -> (HTTPURLResponse, Data))?

    override class func canInit(with request: URLRequest) -> Bool {
        true
    }

    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        request
    }

    override func startLoading() {
        guard let handler = MockURLProtocol.requestHandler else {
            client?.urlProtocol(self, didFailWithError: NetworkError.invalidResponse)
            return
        }

        do {
            let (response, data) = try handler(request)
            client?.urlProtocol(self, didReceive: response, cacheStoragePolicy: .notAllowed)
            client?.urlProtocol(self, didLoad: data)
            client?.urlProtocolDidFinishLoading(self)
        } catch {
            client?.urlProtocol(self, didFailWithError: error)
        }
    }

    override func stopLoading() { }
}

// MARK: - Test Helper

extension URLSession {
    static var mock: URLSession {
        let config = URLSessionConfiguration.ephemeral
        config.protocolClasses = [MockURLProtocol.self]
        return URLSession(configuration: config)
    }
}
```

### 테스트 예시

```swift
import XCTest
@testable import MyApp

final class UserAPIClientTests: XCTestCase {

    private var sut: UserAPIClient!
    private var mockNetwork: MockNetworkService!

    override func setUp() {
        mockNetwork = MockNetworkService()
        sut = UserAPIClient(networkService: mockNetwork)
    }

    override func tearDown() {
        sut = nil
        mockNetwork = nil
    }

    func test_getUser_사용자반환() async throws {
        // Given
        let expectedUser = User(
            id: "123",
            name: "홍길동",
            email: "hong@test.com",
            avatarUrl: nil,
            createdAt: nil
        )
        mockNetwork.setResponse(expectedUser)

        // When
        let user = try await sut.getUser(id: "123")

        // Then
        XCTAssertEqual(user.id, expectedUser.id)
        XCTAssertEqual(user.name, expectedUser.name)
        XCTAssertEqual(mockNetwork.requestCallCount, 1)
    }

    func test_getUser_네트워크오류_에러발생() async {
        // Given
        mockNetwork.setError(.noInternetConnection)

        // When & Then
        do {
            _ = try await sut.getUser(id: "123")
            XCTFail("에러가 발생해야 합니다")
        } catch let error as NetworkError {
            XCTAssertEqual(error, .noInternetConnection)
        } catch {
            XCTFail("예상치 못한 에러 타입")
        }
    }

    func test_getUsers_페이지네이션() async throws {
        // Given
        let response = PaginatedResponse(
            data: [User.mock, User.mock],
            page: 1,
            totalPages: 3,
            totalCount: 50
        )
        mockNetwork.setResponse(response)

        // When
        let result = try await sut.getUsers(page: 1, limit: 20)

        // Then
        XCTAssertEqual(result.data.count, 2)
        XCTAssertTrue(result.hasNextPage)
    }
}

// MARK: - Mock Data

extension User {
    static var mock: User {
        User(
            id: UUID().uuidString,
            name: "테스트 사용자",
            email: "test@example.com",
            avatarUrl: nil,
            createdAt: Date()
        )
    }
}
```

---

## 디버깅 도구

### Request/Response Logger

```swift
// MARK: - Network Logger

final class NetworkLogger {

    static let shared = NetworkLogger()

    var isEnabled = true

    func logRequest(_ request: URLRequest) {
        guard isEnabled else { return }

        print("""

        ═══════════════════════════════════════════════════════════
        🌐 REQUEST
        ═══════════════════════════════════════════════════════════
        URL: \(request.url?.absoluteString ?? "nil")
        Method: \(request.httpMethod ?? "nil")
        Headers: \(request.allHTTPHeaderFields ?? [:])
        Body: \(bodyString(request.httpBody))
        ═══════════════════════════════════════════════════════════

        """)
    }

    func logResponse(_ response: URLResponse?, data: Data?, error: Error?) {
        guard isEnabled else { return }

        let statusCode = (response as? HTTPURLResponse)?.statusCode ?? 0
        let statusEmoji = (200...299).contains(statusCode) ? "✅" : "❌"

        print("""

        ═══════════════════════════════════════════════════════════
        \(statusEmoji) RESPONSE (\(statusCode))
        ═══════════════════════════════════════════════════════════
        URL: \(response?.url?.absoluteString ?? "nil")
        Error: \(error?.localizedDescription ?? "nil")
        Data: \(prettyPrintedJSON(data))
        ═══════════════════════════════════════════════════════════

        """)
    }

    private func bodyString(_ data: Data?) -> String {
        guard let data = data else { return "nil" }
        return prettyPrintedJSON(data)
    }

    private func prettyPrintedJSON(_ data: Data?) -> String {
        guard let data = data else { return "nil" }

        if let json = try? JSONSerialization.jsonObject(with: data),
           let prettyData = try? JSONSerialization.data(withJSONObject: json, options: .prettyPrinted),
           let prettyString = String(data: prettyData, encoding: .utf8) {
            return prettyString
        }

        return String(data: data, encoding: .utf8) ?? "디코딩 불가"
    }
}
```

### cURL 변환

```swift
extension URLRequest {

    var curlString: String {
        var components = ["curl -v"]

        if let method = httpMethod, method != "GET" {
            components.append("-X \(method)")
        }

        if let headers = allHTTPHeaderFields {
            for (key, value) in headers {
                components.append("-H '\(key): \(value)'")
            }
        }

        if let body = httpBody, let bodyString = String(data: body, encoding: .utf8) {
            components.append("-d '\(bodyString)'")
        }

        if let url = url?.absoluteString {
            components.append("'\(url)'")
        }

        return components.joined(separator: " \\\n\t")
    }
}

// 사용
// print(request.curlString)
```

---

## 환경 설정

### API 환경 관리

```swift
// MARK: - API Environment

enum APIEnvironment {
    case development
    case staging
    case production

    var baseURL: URL {
        switch self {
        case .development:
            return URL(string: "https://dev-api.example.com")!
        case .staging:
            return URL(string: "https://staging-api.example.com")!
        case .production:
            return URL(string: "https://api.example.com")!
        }
    }

    var apiKey: String {
        switch self {
        case .development:
            return "dev-api-key"
        case .staging:
            return "staging-api-key"
        case .production:
            return "prod-api-key"
        }
    }

    static var current: APIEnvironment {
        #if DEBUG
        return .development
        #else
        return .production
        #endif
    }
}

// MARK: - Configuration

struct APIConfiguration {

    static let shared = APIConfiguration()

    let environment: APIEnvironment
    let timeout: TimeInterval
    let maxRetries: Int

    private init() {
        environment = APIEnvironment.current
        timeout = 30.0
        maxRetries = 3
    }

    var baseURL: URL {
        environment.baseURL
    }
}
```

---

## 체크리스트

### 구현 전

- [ ] API 문서 확인 (엔드포인트, 요청/응답 형식)
- [ ] 인증 방식 결정 (Bearer, API Key, OAuth)
- [ ] 에러 응답 형식 확인

### 구현

- [ ] Endpoint 프로토콜 정의
- [ ] 요청/응답 모델 정의 (Codable)
- [ ] NetworkService 구현
- [ ] 에러 처리 및 변환
- [ ] 인증 토큰 관리

### 품질

- [ ] 네트워크 모니터링 추가
- [ ] 재시도 로직 구현
- [ ] 캐싱 전략 적용
- [ ] 로깅 추가 (디버그 빌드)

### 테스트

- [ ] Mock NetworkService 작성
- [ ] 성공 케이스 테스트
- [ ] 에러 케이스 테스트
- [ ] 타임아웃 테스트

---

## 서드파티 라이브러리

### Alamofire (선택적)

```swift
// Package.swift에 추가
// .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.8.0")

import Alamofire

final class AlamofireNetworkService {

    private let session: Session

    init() {
        let interceptor = AuthInterceptor()
        session = Session(interceptor: interceptor)
    }

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        try await session.request(
            endpoint.url,
            method: HTTPMethod(rawValue: endpoint.method.rawValue)!,
            headers: HTTPHeaders(endpoint.headers ?? [:])
        )
        .validate()
        .serializingDecodable(T.self)
        .value
    }
}

// Auth Interceptor
final class AuthInterceptor: RequestInterceptor {

    func adapt(
        _ urlRequest: URLRequest,
        for session: Session,
        completion: @escaping (Result<URLRequest, Error>) -> Void
    ) {
        var request = urlRequest

        Task {
            if let token = try? await TokenManager.shared.getValidToken() {
                request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
            }
            completion(.success(request))
        }
    }

    func retry(
        _ request: Request,
        for session: Session,
        dueTo error: Error,
        completion: @escaping (RetryResult) -> Void
    ) {
        guard let response = request.task?.response as? HTTPURLResponse,
              response.statusCode == 401 else {
            completion(.doNotRetry)
            return
        }

        // 토큰 갱신 후 재시도
        completion(.retryWithDelay(1.0))
    }
}
```

### 권장 사항

| 상황 | 권장 |
|------|------|
| 단순 API | URLSession (내장) |
| 복잡한 인터셉터 필요 | Alamofire |
| 이미지 다운로드/캐싱 | Kingfisher, SDWebImage |
| WebSocket | URLSessionWebSocketTask, Starscream |
| GraphQL | Apollo iOS |

---

## 흔한 실수

### 1. 메인 스레드 블로킹

```swift
// ❌ 잘못된 예
func loadData() {
    let data = try! Data(contentsOf: url)  // 메인 스레드 블로킹
}

// ✅ 올바른 예
func loadData() async throws {
    let (data, _) = try await URLSession.shared.data(from: url)
}
```

### 2. 강한 순환 참조

```swift
// ❌ 잘못된 예
networkService.fetch { result in
    self.handleResult(result)  // 강한 참조
}

// ✅ 올바른 예
networkService.fetch { [weak self] result in
    self?.handleResult(result)
}
```

### 3. 에러 무시

```swift
// ❌ 잘못된 예
let user = try? await apiClient.getUser(id: "123")

// ✅ 올바른 예
do {
    let user = try await apiClient.getUser(id: "123")
} catch {
    handleError(error)
}
```

### 4. JSON 키 불일치

```swift
// ❌ 서버 응답: {"user_name": "John"}
struct User: Codable {
    let userName: String  // 디코딩 실패
}

// ✅ 올바른 예
struct User: Codable {
    let userName: String
}

// JSONDecoder 설정
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

### 5. 옵셔널 처리 누락

```swift
// ❌ 서버가 null을 반환할 수 있는 경우
struct User: Codable {
    let avatarUrl: String  // null이면 디코딩 실패
}

// ✅ 올바른 예
struct User: Codable {
    let avatarUrl: String?
}
```

---

## 참고 자료

- [URLSession Documentation](https://developer.apple.com/documentation/foundation/urlsession)
- [Swift Concurrency](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/)
- [Alamofire](https://github.com/Alamofire/Alamofire)
- [Kingfisher](https://github.com/onevcat/Kingfisher)
