---
name: ios-i18n-guide
description: iOS/SwiftUI 국제화(i18n) 구현 가이드 (String Catalogs, 복수형, RTL)
category: apple
tags: [swiftui, ios, i18n, localization, string-catalogs]
---

# iOS/SwiftUI 국제화(i18n) 가이드

## 개요

iOS/SwiftUI 앱의 다국어 지원을 위한 국제화(i18n) 구현 가이드. Xcode 15+의 String Catalogs(.xcstrings) 방식 사용.

---

## String Catalogs 설정

### 파일 생성

1. File > New > File > String Catalog 선택
2. `Localizable.xcstrings` 이름으로 생성
3. 프로젝트 타겟에 추가 확인

### 언어 추가

1. String Catalog 파일 선택
2. 하단 "+" 버튼 클릭
3. 언어 선택 (Korean, Japanese, Chinese 등)

### 빌드 시 자동 추출

1. 코드에 문자열 작성
2. 빌드 실행 (Cmd + B)
3. String Catalog에 새 문자열 자동 추가됨
4. 각 언어별 번역 입력

---

## 기본 사용법

### SwiftUI Text (자동 로컬라이즈)

```swift
// 문자열 리터럴은 자동으로 로컬라이즈됨
Text("Hello, World!")
Button("Save") { }
Label("Settings", systemImage: "gear")
TextField("Enter name", text: $name)
```

### 명시적 키 사용

```swift
// 키 기반 조회
Text("welcome_message")
Text("login_button_title")
```

### String 변환

```swift
// String 타입이 필요할 때
let message = String(localized: "welcome_message")
let error = String(localized: "error_network")

// 변수에 저장 후 사용
let buttonTitle = String(localized: "save_button")
print(buttonTitle)
```

### LocalizedStringKey

```swift
struct ContentView: View {
    // LocalizedStringKey 타입으로 선언
    let titleKey: LocalizedStringKey = "welcome_title"

    var body: some View {
        Text(titleKey)
    }
}
```

---

## 변수 포함 문자열

### String Interpolation

```swift
// 코드
Text("Hello, \(userName)!")
Text("You have \(count) items")
Text("Price: \(price, format: .currency(code: "KRW"))")

// String Catalog에서 자동 인식:
// "Hello, %@!"
// "You have %lld items"
// "Price: %@"
```

### 여러 변수

```swift
// 코드
Text("From \(startDate, style: .date) to \(endDate, style: .date)")

// String Catalog:
// "From %@ to %@"
// 한국어: "%@부터 %@까지"
```

### String(localized:)로 변수 사용

```swift
// 방법 1: interpolation
let greeting = String(localized: "Hello, \(userName)!")

// 방법 2: format 사용
let message = String(format: String(localized: "welcome_format"), userName, itemCount)
```

---

## 복수형 (Pluralization)

### 설정 방법

1. String Catalog에서 키 선택
2. 우클릭 > "Vary by Plural" 체크
3. 각 복수형 옵션에 번역 입력

### 복수형 옵션

| 옵션 | 설명 | 예시 (한국어) |
|------|------|-------------|
| zero | 0일 때 | "항목 없음" |
| one | 1일 때 | "1개 항목" |
| two | 2일 때 (아랍어 등) | - |
| few | 소수일 때 (러시아어 등) | - |
| many | 다수일 때 | - |
| other | 기본값 (필수) | "%lld개 항목" |

### 코드 예시

```swift
// 코드
Text("item_count \(count)")

// String Catalog 설정:
// Key: "item_count %lld"
// - zero: "항목 없음"
// - one: "1개 항목"
// - other: "%lld개 항목"

// 결과:
// count = 0 -> "항목 없음"
// count = 1 -> "1개 항목"
// count = 5 -> "5개 항목"
```

---

## 기기별 분기

### 설정 방법

1. String Catalog에서 키 선택
2. 우클릭 > "Vary by Device" 체크
3. 각 기기별 번역 입력

### 기기 옵션

```swift
// String Catalog 설정:
// Key: "tap_to_continue"
// - iPhone: "탭하여 계속"
// - iPad: "탭하거나 클릭하여 계속"
// - Mac: "클릭하여 계속"
// - Apple Watch: "눌러서 계속"
// - Apple TV: "선택하여 계속"
```

---

## 숫자, 날짜, 통화 포맷팅

### 숫자

```swift
let number = 1234567.89

// SwiftUI - 로케일 자동 적용
Text(number, format: .number)
// ko_KR: "1,234,567.89"
// de_DE: "1.234.567,89"

// 소수점 자릿수 지정
Text(number, format: .number.precision(.fractionLength(2)))

// 퍼센트
Text(0.85, format: .percent)  // "85%"
```

### 통화

```swift
let price = 29900.0

// 통화 코드 지정
Text(price, format: .currency(code: "KRW"))  // "₩29,900"
Text(price, format: .currency(code: "USD"))  // "$29,900.00"
Text(price, format: .currency(code: "JPY"))  // "¥29,900"

// Decimal 타입 권장 (정확한 금액 표시)
let exactPrice = Decimal(29900)
Text(exactPrice, format: .currency(code: "KRW"))
```

### 날짜

```swift
let date = Date()

// 스타일 기반
Text(date, style: .date)      // "2024년 1월 15일"
Text(date, style: .time)      // "오후 3:30"
Text(date, style: .relative)  // "2시간 전"
Text(date, style: .timer)     // 타이머 형식

// 커스텀 포맷
Text(date, format: .dateTime.year().month().day())
Text(date, format: .dateTime.hour().minute())
Text(date, format: .dateTime.weekday(.wide))  // "월요일"

// 날짜 범위
Text(startDate...endDate)
```

### 단위 (Measurement)

```swift
// 거리
let distance = Measurement(value: 5.5, unit: UnitLength.kilometers)
Text(distance, format: .measurement(width: .wide))        // "5.5킬로미터"
Text(distance, format: .measurement(width: .abbreviated)) // "5.5km"

// 무게
let weight = Measurement(value: 75, unit: UnitMass.kilograms)
Text(weight, format: .measurement(width: .abbreviated))   // "75kg"

// 온도
let temp = Measurement(value: 25, unit: UnitTemperature.celsius)
Text(temp, format: .measurement(width: .abbreviated))     // "25°C"
```

---

## Info.plist 로컬라이제이션

### InfoPlist.xcstrings 생성

1. File > New > File > String Catalog
2. `InfoPlist.xcstrings` 이름으로 생성

### 앱 이름 로컬라이즈

```
// InfoPlist.xcstrings에 추가
Key: "CFBundleDisplayName"
- English: "My App"
- Korean: "나의 앱"
- Japanese: "マイアプリ"
```

### 권한 메시지 로컬라이즈

```
// 카메라
Key: "NSCameraUsageDescription"
- English: "Camera access is needed to take photos."
- Korean: "사진 촬영을 위해 카메라 접근이 필요합니다."

// 사진 라이브러리
Key: "NSPhotoLibraryUsageDescription"
- English: "Photo library access is needed to save images."
- Korean: "이미지 저장을 위해 사진 라이브러리 접근이 필요합니다."
```

---

## SwiftUI 로컬라이제이션 패턴

### 에러 메시지 로컬라이즈

```swift
enum AppError: LocalizedError {
    case networkUnavailable
    case invalidInput
    case unauthorized
    case serverError(code: Int)

    var errorDescription: String? {
        switch self {
        case .networkUnavailable:
            return String(localized: "error_network_unavailable")
        case .invalidInput:
            return String(localized: "error_invalid_input")
        case .unauthorized:
            return String(localized: "error_unauthorized")
        case .serverError(let code):
            return String(localized: "error_server \(code)")
        }
    }
}
```

### 로컬라이즈된 문자열 헬퍼

```swift
enum L10n {
    // 일반
    static let ok = String(localized: "common_ok")
    static let cancel = String(localized: "common_cancel")
    static let save = String(localized: "common_save")
    static let delete = String(localized: "common_delete")
    static let edit = String(localized: "common_edit")

    // 동적 문자열
    static func greeting(name: String) -> String {
        String(localized: "greeting \(name)")
    }

    static func itemCount(_ count: Int) -> String {
        String(localized: "item_count \(count)")
    }
}

// 사용
Text(L10n.greeting(name: userName))
Button(L10n.save) { }
```

---

## RTL (Right-to-Left) 지원

### 자동 RTL 레이아웃

SwiftUI는 아랍어, 히브리어 등 RTL 언어 자동 지원.

```swift
// leading/trailing 사용 (RTL 자동 대응)
HStack {
    Text("Label")
    Spacer()
    Text("Value")
}
.padding(.leading, 16)  // RTL에서 자동으로 오른쪽이 됨
```

### 명시적 방향 고정

```swift
// RTL 무시하고 항상 LTR
HStack {
    Text("Always Left to Right")
}
.environment(\.layoutDirection, .leftToRight)

// 전화번호, 코드 등 항상 LTR
Text("+82 10-1234-5678")
    .environment(\.layoutDirection, .leftToRight)
```

---

## 테스트

### SwiftUI Preview에서 테스트

```swift
#Preview("Korean") {
    ContentView()
        .environment(\.locale, Locale(identifier: "ko"))
}

#Preview("English") {
    ContentView()
        .environment(\.locale, Locale(identifier: "en"))
}

#Preview("Arabic RTL") {
    ContentView()
        .environment(\.locale, Locale(identifier: "ar"))
        .environment(\.layoutDirection, .rightToLeft)
}
```

---

## 체크리스트

### 초기 설정
- [ ] Localizable.xcstrings 생성
- [ ] InfoPlist.xcstrings 생성
- [ ] 지원할 언어 목록 결정 및 추가

### 코드 작성
- [ ] 모든 사용자 노출 문자열 로컬라이즈
- [ ] 하드코딩된 문자열 검색
- [ ] 복수형 처리 필요한 곳 확인
- [ ] 날짜/숫자/통화에 포맷터 사용
- [ ] 이미지/아이콘 RTL 대응 확인

### 배포 전
- [ ] 모든 언어 번역 완료 (New 상태 없음)
- [ ] Needs Review 항목 검토 완료
- [ ] 각 언어별 스크린샷 검수
- [ ] TestFlight에서 실제 기기 테스트

---

## 흔한 실수와 해결

### 문자열이 번역되지 않음

```swift
// String 변수는 자동 로컬라이즈 안 됨 - 주의
let title: String = "Hello"
Text(title)  // 로컬라이즈 안 됨

// 리터럴 직접 사용 - 권장
Text("Hello")

// String(localized:) 사용 - 권장
let title = String(localized: "hello_key")
Text(title)
```

### 키 이름 컨벤션

```
{화면}_{요소}_{설명}

예시:
- home_title
- login_button_submit
- login_field_email_placeholder
- settings_section_account
- alert_logout_title
```
