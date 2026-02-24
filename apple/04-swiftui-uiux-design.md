---
name: swiftui-uiux-design
description: SwiftUI UI/UX 디자인 원칙, Apple HIG 기반 가이드
category: apple
tags: [swiftui, ios, macos, uiux, design, hig, accessibility]
---

# SwiftUI UI/UX 디자인 스킬

## 개요
SwiftUI 앱 개발 시 적용해야 하는 UI/UX 디자인 원칙을 정의합니다.
Apple Human Interface Guidelines(HIG)를 기반으로 합니다.

---

## Apple Human Interface Guidelines 참조

### MCP를 통한 HIG 문서 참조
UI/UX 설계 시 반드시 Apple Human Interface Guidelines를 참고해야 합니다.

```
web_fetch: https://developer.apple.com/kr/design/human-interface-guidelines/
```

### 주요 참고 섹션
- **Platforms > iOS / macOS**: 플랫폼별 고유 가이드라인
- **Components**: 표준 UI 컴포넌트 사용법
- **Patterns**: 일반적인 UI 패턴
- **Inputs**: 키보드, 마우스, 트랙패드, 터치 입력 처리
- **Technologies > Accessibility**: 접근성 구현

---

## 디자인 핵심 원칙

### 네이티브 경험 우선
- 시스템 표준 컴포넌트 우선 사용
- 커스텀 컴포넌트는 시스템 동작과 일관되게 구현
- 플랫폼 사용자가 기대하는 동작 패턴 준수

### 시스템 통합
```swift
// 시스템 색상 사용
Text("제목")
    .foregroundStyle(.primary)
    .background(.background)

// SF Symbols 활용
Image(systemName: "folder.fill")
    .symbolRenderingMode(.hierarchical)

// 시스템 폰트 사용
Text("본문")
    .font(.body)
```

### 다크 모드 / 라이트 모드 지원
```swift
// 자동 색상 적응
struct AdaptiveCard: View {
    @Environment(\.colorScheme) var colorScheme

    var body: some View {
        RoundedRectangle(cornerRadius: 8)
            .fill(Color(.windowBackgroundColor))
            .overlay(
                // 시스템 색상은 자동으로 모드에 적응
                Text("내용")
                    .foregroundStyle(.primary)
            )
    }
}

// Assets에서 색상 세트 정의
// Assets.xcassets에서 Any Appearance / Dark 모두 설정
```

---

## 창 및 레이아웃

### 창 관리
```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .windowStyle(.automatic)
        .windowResizability(.contentSize)  // 콘텐츠에 맞춤
        // 또는
        .windowResizability(.contentMinSize)  // 최소 크기 지정

        Settings {
            SettingsView()
        }
    }
}
```

### 반응형 레이아웃
```swift
struct ResponsiveLayout: View {
    var body: some View {
        GeometryReader { geometry in
            if geometry.size.width > 600 {
                // 넓은 화면: 사이드바 레이아웃
                HStack(spacing: 0) {
                    SidebarView()
                        .frame(width: 250)
                    ContentView()
                }
            } else {
                // 좁은 화면: 탭 레이아웃
                TabView {
                    // ...
                }
            }
        }
    }
}
```

### NavigationSplitView 활용
```swift
struct MainView: View {
    @State private var selectedItem: Item?

    var body: some View {
        NavigationSplitView {
            // 사이드바
            List(items, selection: $selectedItem) { item in
                NavigationLink(value: item) {
                    ItemRow(item: item)
                }
            }
            .navigationSplitViewColumnWidth(min: 200, ideal: 250, max: 300)
        } detail: {
            // 상세 뷰
            if let item = selectedItem {
                ItemDetailView(item: item)
            } else {
                Text("항목을 선택하세요")
                    .foregroundStyle(.secondary)
            }
        }
    }
}
```

---

## 키보드 및 메뉴 통합

### 키보드 단축키
```swift
struct ContentView: View {
    var body: some View {
        VStack {
            // 콘텐츠
        }
        .keyboardShortcut("n", modifiers: [.command])  // Cmd+N
    }
}

// 커스텀 단축키 정의
extension KeyboardShortcut {
    static let refresh = KeyboardShortcut("r", modifiers: .command)
    static let delete = KeyboardShortcut(.delete, modifiers: .command)
}
```

### 메뉴바 통합
```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .commands {
            // 기존 메뉴 수정
            CommandGroup(replacing: .newItem) {
                Button("새 문서") {
                    // 액션
                }
                .keyboardShortcut("n", modifiers: .command)
            }

            // 새 메뉴 추가
            CommandMenu("도구") {
                Button("분석 실행") {
                    // 액션
                }
                .keyboardShortcut("a", modifiers: [.command, .shift])

                Divider()

                Button("설정 초기화") {
                    // 액션
                }
            }
        }
    }
}
```

### 컨텍스트 메뉴
```swift
struct ItemRow: View {
    let item: Item

    var body: some View {
        Text(item.name)
            .contextMenu {
                Button("편집") { /* 액션 */ }
                Button("복제") { /* 액션 */ }
                Divider()
                Button("삭제", role: .destructive) { /* 액션 */ }
            }
    }
}
```

---

## 접근성 (Accessibility)

### 기본 접근성 지원
```swift
// 접근성 레이블 추가
Image(systemName: "star.fill")
    .accessibilityLabel("즐겨찾기")
    .accessibilityHint("즐겨찾기에 추가합니다")
    .accessibilityAddTraits(.isButton)

// 접근성 값 설정
Slider(value: $volume, in: 0...100)
    .accessibilityValue("\(Int(volume))퍼센트")

// 그룹화
VStack {
    Text("사용자 정보")
    Text(user.name)
    Text(user.email)
}
.accessibilityElement(children: .combine)
```

### VoiceOver 지원
```swift
struct AccessibleCard: View {
    let item: Item

    var body: some View {
        VStack {
            Image(item.imageName)
            Text(item.title)
            Text(item.description)
        }
        .accessibilityElement(children: .combine)
        .accessibilityLabel("\(item.title), \(item.description)")
        .accessibilityHint("두 번 탭하여 상세 정보 보기")
    }
}
```

### 동적 타입 지원
```swift
// 시스템 폰트 사용 (자동 크기 조절)
Text("본문 텍스트")
    .font(.body)

// 커스텀 폰트도 동적 타입 지원
Text("커스텀 텍스트")
    .font(.custom("MyFont", size: 16, relativeTo: .body))
```

---

## 애니메이션 및 전환

### 시스템 애니메이션 활용
```swift
struct AnimatedView: View {
    @State private var isExpanded = false

    var body: some View {
        VStack {
            Button("토글") {
                withAnimation(.spring(response: 0.3, dampingFraction: 0.7)) {
                    isExpanded.toggle()
                }
            }

            if isExpanded {
                DetailView()
                    .transition(.asymmetric(
                        insertion: .scale.combined(with: .opacity),
                        removal: .opacity
                    ))
            }
        }
    }
}
```

### 접근성을 고려한 애니메이션
```swift
struct MotionSafeView: View {
    @Environment(\.accessibilityReduceMotion) var reduceMotion

    var body: some View {
        Content()
            .animation(reduceMotion ? .none : .default, value: someValue)
    }
}
```

---

## 피드백 및 상태 표시

### 로딩 상태
```swift
struct LoadingView: View {
    var body: some View {
        ProgressView("로딩 중...")
            .progressViewStyle(.circular)
    }
}

// 진행률 표시
ProgressView(value: progress, total: 100) {
    Text("다운로드 중")
} currentValueLabel: {
    Text("\(Int(progress))%")
}
```

### 알림 및 경고
```swift
struct ContentView: View {
    @State private var showAlert = false

    var body: some View {
        Button("삭제") {
            showAlert = true
        }
        .alert("삭제 확인", isPresented: $showAlert) {
            Button("취소", role: .cancel) { }
            Button("삭제", role: .destructive) {
                // 삭제 실행
            }
        } message: {
            Text("이 항목을 삭제하시겠습니까? 이 작업은 취소할 수 없습니다.")
        }
    }
}
```

---

## 참고 자료

- [Apple Human Interface Guidelines](https://developer.apple.com/kr/design/human-interface-guidelines/)
- [macOS Design Themes](https://developer.apple.com/design/human-interface-guidelines/platforms/designing-for-macos)
- [SF Symbols](https://developer.apple.com/sf-symbols/)
- [Accessibility Guidelines](https://developer.apple.com/accessibility/)
