---
name: storekit-iap
description: StoreKit 2 기반 iOS/watchOS 인앱결제 및 구독 구현 가이드
category: apple
tags: [swiftui, ios, watchos, storekit, iap, subscription]
---

# iOS/watchOS 인앱결제 구현 가이드

## 개요

StoreKit 2 기반 인앱결제 구현을 위한 가이드. iOS 15+, watchOS 8+ 지원.

## 상품 유형

| 유형 | 용도 | 예시 |
|------|------|------|
| **Consumable** | 1회성 소모 아이템 | 게임 코인, 크레딧 |
| **Non-Consumable** | 영구 구매 | 프리미엄 잠금해제, 광고제거 |
| **Auto-Renewable** | 자동갱신 구독 | 월간/연간 구독 |
| **Non-Renewing** | 수동갱신 구독 | 시즌패스 |

---

## 핵심 구현 코드

### StoreManager 전체 구현

```swift
import StoreKit

enum StoreError: Error {
    case failedVerification
    case productNotFound
    case purchaseFailed
}

@MainActor
class StoreManager: ObservableObject {
    @Published private(set) var products: [Product] = []
    @Published private(set) var purchasedProductIDs = Set<String>()

    private let productIDs: Set<String>
    private var transactionListener: Task<Void, Error>?

    init(productIDs: Set<String>) {
        self.productIDs = productIDs
        transactionListener = listenForTransactions()

        Task {
            await loadProducts()
            await updatePurchasedProducts()
        }
    }

    deinit {
        transactionListener?.cancel()
    }

    // MARK: - 상품 로드

    func loadProducts() async {
        do {
            products = try await Product.products(for: productIDs)
                .sorted { $0.price < $1.price }
        } catch {
            print("상품 로드 실패: \(error)")
        }
    }

    // MARK: - 구매 처리

    func purchase(_ product: Product) async throws -> Transaction? {
        let result = try await product.purchase()

        switch result {
        case .success(let verification):
            let transaction = try checkVerified(verification)
            await updatePurchasedProducts()
            await transaction.finish()
            return transaction

        case .userCancelled, .pending:
            return nil

        @unknown default:
            return nil
        }
    }

    private func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .unverified:
            throw StoreError.failedVerification
        case .verified(let safe):
            return safe
        }
    }

    // MARK: - 트랜잭션 리스너

    private func listenForTransactions() -> Task<Void, Error> {
        Task.detached {
            for await result in Transaction.updates {
                do {
                    let transaction = try await self.checkVerified(result)
                    await self.updatePurchasedProducts()
                    await transaction.finish()
                } catch {
                    print("트랜잭션 검증 실패: \(error)")
                }
            }
        }
    }

    // MARK: - 구매 상태 확인

    func updatePurchasedProducts() async {
        for await result in Transaction.currentEntitlements {
            guard case .verified(let transaction) = result else { continue }

            if transaction.revocationDate == nil {
                purchasedProductIDs.insert(transaction.productID)
            } else {
                purchasedProductIDs.remove(transaction.productID)
            }
        }
    }

    func isPurchased(_ productID: String) -> Bool {
        purchasedProductIDs.contains(productID)
    }

    // MARK: - 구매 복원

    func restore() async throws {
        try await AppStore.sync()
        await updatePurchasedProducts()
    }
}
```

### SwiftUI View 연동

```swift
struct StoreView: View {
    @StateObject private var store = StoreManager(
        productIDs: ["com.app.premium", "com.app.coins100"]
    )

    var body: some View {
        List {
            ForEach(store.products) { product in
                HStack {
                    VStack(alignment: .leading) {
                        Text(product.displayName)
                            .font(.headline)
                        Text(product.description)
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }

                    Spacer()

                    if store.isPurchased(product.id) {
                        Image(systemName: "checkmark.circle.fill")
                            .foregroundColor(.green)
                    } else {
                        Button(product.displayPrice) {
                            Task {
                                try? await store.purchase(product)
                            }
                        }
                        .buttonStyle(.borderedProminent)
                    }
                }
            }

            Section {
                Button("구매 복원") {
                    Task {
                        try? await store.restore()
                    }
                }
            }
        }
    }
}
```

---

## 구독 구현 (Auto-Renewable)

### SubscriptionManager

```swift
@MainActor
class SubscriptionManager: ObservableObject {
    @Published var subscriptions: [Product] = []
    @Published var currentSubscription: Product?
    @Published var isSubscribed = false

    private let groupID: String
    private let productIDs: Set<String>

    init(groupID: String, productIDs: Set<String>) {
        self.groupID = groupID
        self.productIDs = productIDs

        Task {
            await loadSubscriptions()
            await updateSubscriptionStatus()
        }
    }

    func loadSubscriptions() async {
        do {
            let products = try await Product.products(for: productIDs)
            subscriptions = products
                .filter { $0.type == .autoRenewable }
                .sorted { $0.price < $1.price }
        } catch {
            print("구독 상품 로드 실패: \(error)")
        }
    }

    func updateSubscriptionStatus() async {
        do {
            let statuses = try await Product.SubscriptionInfo.status(for: groupID)

            for status in statuses {
                switch status.state {
                case .subscribed, .inGracePeriod, .inBillingRetryPeriod:
                    isSubscribed = true
                    if case .verified(let transaction) = status.transaction {
                        currentSubscription = subscriptions.first {
                            $0.id == transaction.productID
                        }
                    }
                    return
                default:
                    continue
                }
            }
            isSubscribed = false
            currentSubscription = nil
        } catch {
            print("구독 상태 확인 실패: \(error)")
        }
    }

    func purchase(_ product: Product) async throws {
        let result = try await product.purchase()

        if case .success(let verification) = result,
           case .verified(let transaction) = verification {
            await transaction.finish()
            await updateSubscriptionStatus()
        }
    }
}
```

### 무료 체험 확인

```swift
extension Product {
    var hasFreeTrial: Bool {
        subscription?.introductoryOffer?.paymentMode == .freeTrial
    }

    var freeTrialDays: Int? {
        guard let offer = subscription?.introductoryOffer,
              offer.paymentMode == .freeTrial else { return nil }

        let period = offer.period
        switch period.unit {
        case .day: return period.value
        case .week: return period.value * 7
        case .month: return period.value * 30
        case .year: return period.value * 365
        @unknown default: return nil
        }
    }

    func isEligibleForFreeTrial() async -> Bool {
        await subscription?.isEligibleForIntroOffer ?? false
    }
}
```

### 구독 관리 페이지 열기

```swift
// iOS 15+
func openSubscriptionManagement() async {
    if let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene {
        try? await AppStore.showManageSubscriptions(in: windowScene)
    }
}
```

---

## StoreKit 2 API 레퍼런스

### Product

```swift
struct Product: Identifiable {
    let id: String              // Product ID
    let displayName: String     // 현지화된 상품명
    let description: String     // 현지화된 설명
    let price: Decimal          // 가격 (숫자)
    let displayPrice: String    // 현지화된 가격 문자열
    let type: Product.ProductType

    static func products(for identifiers: some Collection<String>) async throws -> [Product]
    func purchase(options: Set<Product.PurchaseOption> = []) async throws -> Product.PurchaseResult
}
```

### Transaction

```swift
struct Transaction {
    let id: UInt64
    let productID: String
    let purchaseDate: Date
    let expirationDate: Date?
    let revocationDate: Date?
    let isUpgraded: Bool

    static var currentEntitlements: Transaction.Entitlements { get }
    static var updates: Transaction.Updates { get }

    func finish() async
}
```

---

## watchOS 특이사항

- **watchOS 6.2+** 필요 (독립 결제 지원)
- iPhone 없이 Watch 단독 결제 가능
- 화면 크기에 맞는 간결한 UI 필수
- 독립 실행형(standalone) 앱 설정 필요
- Info.plist에서 `WKRunsIndependentlyOfCompanionApp = YES`

---

## App Store Connect 설정

1. 앱 > 인앱 구입 > 관리 메뉴 진입
2. 상품 유형 선택 및 Product ID 설정 (예: `com.yourapp.premium`)
3. 가격 및 지역 설정
4. 심사용 스크린샷 및 설명 추가
5. Sandbox 테스터 계정 생성 (Users and Access > Sandbox)

---

## 테스트 방법

### Xcode StoreKit Configuration (추천)

1. File > New > File > StoreKit Configuration File
2. 상품 추가 (+ 버튼)
3. Scheme > Edit Scheme > Run > Options > StoreKit Configuration 선택
4. 시뮬레이터에서 바로 테스트 가능

### Sandbox 테스트

1. App Store Connect에서 Sandbox 테스터 생성
2. 테스트 기기에서 기존 Apple ID 로그아웃
3. 앱 내 구매 시 Sandbox 계정으로 로그인

---

## 체크리스트

- [ ] Product ID가 App Store Connect와 정확히 일치
- [ ] StoreManager 초기화 시 transactionListener 설정
- [ ] 모든 트랜잭션에 `finish()` 호출
- [ ] 구매 복원 기능 구현 (App Store 심사 필수)
- [ ] 에러 처리 및 사용자 피드백 UI
- [ ] StoreKit Configuration 또는 Sandbox 테스트 완료
- [ ] 구독의 경우 만료/갱신 상태 처리
