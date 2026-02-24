---
name: claude-ios-project
description: iOS/macOS SwiftUI 프로젝트용 CLAUDE.md 스타터 템플릿
category: templates
tags: [template, ios, swiftui, xcode]
---

# CLAUDE.md — {ProjectName} 개발 가이드라인

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

{ProjectName}은(는) {플랫폼: iOS/watchOS/macOS}에서 실행되는 **{앱 한 줄 설명}** 앱입니다.

### 핵심 기능

1. **{기능 1}**: {설명}
2. **{기능 2}**: {설명}
3. **{기능 3}**: {설명}

---

## Build & Run Commands

```bash
# Build
xcodebuild -scheme {AppScheme} -configuration Debug build

# Run tests
xcodebuild -scheme {AppScheme} -configuration Debug test

# Clean build
xcodebuild -scheme {AppScheme} clean

# Open in Xcode
open {ProjectName}.xcodeproj
```

---

## Architecture

### Tech Stack
- **UI**: SwiftUI ({최소 OS 버전}+)
- **Package Manager**: Swift Package Manager
- **Localization**: String Catalogs (.xcstrings)

### Core Components
- `App/`: SwiftUI App entry point
- `Views/`: SwiftUI views
- `Models/`: Data models
- `Services/`: Business logic
- `Resources/`: Localization files

### Key Patterns
- **MVVM 패턴**: View → ViewModel → Model/Service
- **Protocol 기반 의존성 주입**
- **관심사 분리**: 폴더 단위 모듈화

---

## 스킬 참조

이 프로젝트의 개발 가이드라인은 스킬 파일들로 구성되어 있습니다.
각 작업 수행 시 해당 스킬 파일을 참조하세요.

| 작업 유형 | 참조 스킬 | 설명 |
|----------|----------|------|
| 코드 작성 | `swiftui-coding-rules` | 코딩 규칙 및 SOLID 원칙 |
| UI 구현 | `swiftui-uiux-design` | Apple HIG 기반 UI/UX 가이드 |
| 새 기능 설계 | `swiftui-architecture` | 프로젝트 구조 및 아키텍처 |
| 테스트 작성 | `swiftui-testing` | 테스트 전략 및 작성법 |
| 커밋/PR | `swiftui-git-workflow` | Git 커밋 및 협업 규칙 |
| CLI 빌드 | `xcode-cli` | Xcode CLI 도구 및 자동화 |
| 다국어 지원 | `ios-i18n-guide` | 국제화 가이드 |
| 인앱 구매 | `storekit-iap` | StoreKit IAP 구현 |
| 배포 | `ios-deploy` | App Store 배포 가이드 |

---

## 핵심 원칙 요약

### 코드 품질
- 파일당 100줄 미만
- 평탄한 구조 (중첩 방지)
- 불변 데이터 (`struct`, `let`) 우선
- SOLID 원칙 준수
- 주석은 한글(Korean)로 작성

### UI/UX
- Apple HIG 준수
- 접근성 (Accessibility) 구현 필수
- 다크 모드 / 라이트 모드 지원

### 아키텍처
- MVVM 패턴
- Protocol 기반 의존성 주입
- 관심사 분리

### 문서화
- 한글 주석
- 한글 커밋 메시지
- Conventional Commits 형식

---

## 참고 자료

- [Apple Human Interface Guidelines](https://developer.apple.com/kr/design/human-interface-guidelines/)
- [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui/)
