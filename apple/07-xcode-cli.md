---
name: xcode-cli
description: Xcode CLI 도구 완전 가이드 (xcodebuild, simctl, swift, codesign 등)
category: apple
tags: [xcode, ios, macos, cli, build, simulator]
---

# Xcode CLI 스킬

## 개요
이 스킬은 터미널에서 Xcode 관련 모든 작업을 수행할 수 있는 CLI 명령어들을 제공합니다.
iOS/macOS 앱 개발, iOS 시뮬레이터 관리, 빌드, 테스트, 배포까지 전체 워크플로우를 CLI로 처리합니다.

---

## 사전 요구사항

Xcode 및 Command Line Tools가 설치되어 있어야 합니다:
```bash
# Command Line Tools 설치
xcode-select --install

# 설치 확인
xcode-select -p
```

---

## 1. xcode-select - Xcode 버전 관리

### 현재 Xcode 경로 확인
```bash
xcode-select -p
```

### Xcode 버전 변경
```bash
# 특정 Xcode 버전으로 변경
sudo xcode-select -s /Applications/Xcode.app

# 베타 버전 사용 시
sudo xcode-select -s /Applications/Xcode-beta.app
```

### Command Line Tools 관리
```bash
# 설치
xcode-select --install

# 설정 초기화
sudo xcode-select --reset
```

---

## 2. xcodebuild - 빌드/테스트/아카이브

### 프로젝트 정보 확인
```bash
# 프로젝트의 스킴, 타겟, 설정 확인
xcodebuild -list

# 사용 가능한 SDK 목록
xcodebuild -showsdks

# destination 목록 확인
xcodebuild -showdestinations -scheme <SCHEME_NAME>

# 빌드 설정 확인
xcodebuild -scheme <SCHEME_NAME> -showBuildSettings
```

### macOS 앱 빌드
```bash
# 기본 빌드
xcodebuild -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -configuration Debug \
  build

# Release 빌드
xcodebuild -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -configuration Release \
  build

# 워크스페이스 빌드 (SPM/CocoaPods 사용 시)
xcodebuild -workspace <PROJECT_NAME>.xcworkspace \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  build

# 클린 후 빌드
xcodebuild -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  clean build

# 빌드 결과물 경로 지정
xcodebuild -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -derivedDataPath ./build \
  build
```

### iOS 앱 빌드
```bash
# iOS 시뮬레이터 빌드
xcodebuild -scheme <SCHEME_NAME> \
  -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.0' \
  build

# iOS 기기용 빌드 (Archive 권장)
xcodebuild -scheme <SCHEME_NAME> \
  -destination 'generic/platform=iOS' \
  build
```

### 테스트 실행
```bash
# macOS 테스트
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS'

# iOS 시뮬레이터 테스트
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=iOS Simulator,name=iPhone 15'

# 특정 테스트만 실행
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -only-testing:<TARGET>/<TEST_CLASS>/<TEST_METHOD>

# 특정 테스트 제외
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -skip-testing:<TARGET>/<TEST_CLASS>

# 테스트 결과 저장
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -resultBundlePath ./TestResults.xcresult

# 코드 커버리지 포함
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -enableCodeCoverage YES

# 병렬 테스트
xcodebuild test \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -parallel-testing-enabled YES \
  -maximum-concurrent-test-simulator-destinations 4
```

### 아카이브 생성
```bash
# macOS 앱 아카이브
xcodebuild archive \
  -scheme <SCHEME_NAME> \
  -destination 'platform=macOS' \
  -archivePath ./build/<APP_NAME>.xcarchive

# iOS 앱 아카이브
xcodebuild archive \
  -scheme <SCHEME_NAME> \
  -destination 'generic/platform=iOS' \
  -archivePath ./build/<APP_NAME>.xcarchive
```

### 아카이브 내보내기
```bash
# 앱 내보내기
xcodebuild -exportArchive \
  -archivePath ./build/<APP_NAME>.xcarchive \
  -exportPath ./build/export \
  -exportOptionsPlist ExportOptions.plist
```

---

## 3. xcrun - Xcode 도구 래퍼

### SDK 정보
```bash
# macOS SDK 경로
xcrun --sdk macosx --show-sdk-path

# iOS SDK 경로
xcrun --sdk iphoneos --show-sdk-path

# SDK 버전 확인
xcrun --sdk macosx --show-sdk-version
```

### 도구 경로 확인
```bash
xcrun --find swift
xcrun --find swiftc
xcrun --find clang
xcrun --find xcodebuild
```

### Swift 컴파일러 직접 실행
```bash
# Swift 파일 컴파일
xcrun swiftc main.swift -o main

# 최적화 빌드
xcrun swiftc -O main.swift -o main
```

---

## 4. simctl - iOS 시뮬레이터 제어

### 시뮬레이터 목록 조회
```bash
# 전체 목록
xcrun simctl list

# 디바이스만
xcrun simctl list devices

# 사용 가능한 런타임
xcrun simctl list runtimes

# 디바이스 타입
xcrun simctl list devicetypes

# JSON 형식 출력
xcrun simctl list -j
```

### 시뮬레이터 생성/삭제
```bash
# 새 시뮬레이터 생성
xcrun simctl create "<DEVICE_NAME>" \
  "com.apple.CoreSimulator.SimDeviceType.iPhone-15" \
  "com.apple.CoreSimulator.SimRuntime.iOS-17-0"

# 시뮬레이터 삭제
xcrun simctl delete "<DEVICE_NAME_OR_UDID>"

# 사용 불가능한 시뮬레이터 정리
xcrun simctl delete unavailable
```

### 시뮬레이터 부팅/종료
```bash
# 시뮬레이터 부팅
xcrun simctl boot "<DEVICE_NAME_OR_UDID>"

# 시뮬레이터 종료
xcrun simctl shutdown "<DEVICE_NAME_OR_UDID>"

# 모든 시뮬레이터 종료
xcrun simctl shutdown all

# 시뮬레이터 UI 열기
open -a Simulator
```

### 앱 설치/실행
```bash
# 앱 설치
xcrun simctl install booted <APP_PATH>

# 앱 실행
xcrun simctl launch booted <BUNDLE_ID>

# 앱 실행 (콘솔 출력 포함)
xcrun simctl launch --console booted <BUNDLE_ID>

# 앱 종료
xcrun simctl terminate booted <BUNDLE_ID>

# 앱 제거
xcrun simctl uninstall booted <BUNDLE_ID>
```

### 미디어 캡처
```bash
# 스크린샷
xcrun simctl io booted screenshot screenshot.png

# 비디오 녹화 시작
xcrun simctl io booted recordVideo video.mov
# Ctrl+C로 녹화 중지
```

### URL 및 딥링크
```bash
# URL 열기
xcrun simctl openurl booted "https://example.com"

# 딥링크 열기
xcrun simctl openurl booted "myapp://path/to/content"
```

### 푸시 알림 테스트
```bash
# 푸시 알림 전송
xcrun simctl push booted <BUNDLE_ID> notification.apns

# STDIN으로 전송
echo '{"aps":{"alert":"Hello"}}' | xcrun simctl push booted <BUNDLE_ID> -
```

### 시뮬레이터 상태 설정
```bash
# 시뮬레이터 초기화
xcrun simctl erase "<DEVICE_NAME_OR_UDID>"

# 모든 시뮬레이터 초기화
xcrun simctl erase all

# 위치 설정
xcrun simctl location booted set <LAT>,<LON>
xcrun simctl location booted set 37.7749,-122.4194

# 상태바 오버라이드 (스크린샷용)
xcrun simctl status_bar booted override \
  --time "9:41" \
  --batteryLevel 100 \
  --batteryState charged \
  --cellularBars 4 \
  --wifiBars 3

# 상태바 초기화
xcrun simctl status_bar booted clear

# 다크 모드 설정
xcrun simctl ui booted appearance dark
xcrun simctl ui booted appearance light
```

### 데이터 관리
```bash
# 앱 데이터 경로 확인
xcrun simctl get_app_container booted <BUNDLE_ID> data

# 앱 번들 경로 확인
xcrun simctl get_app_container booted <BUNDLE_ID> app

# 키체인 리셋
xcrun simctl keychain booted reset
```

### 권한 설정
```bash
# 권한 부여
xcrun simctl privacy booted grant <SERVICE> <BUNDLE_ID>

# 권한 거부
xcrun simctl privacy booted revoke <SERVICE> <BUNDLE_ID>

# 권한 초기화
xcrun simctl privacy booted reset <SERVICE> <BUNDLE_ID>

# 모든 권한 초기화
xcrun simctl privacy booted reset all <BUNDLE_ID>
```

**서비스 종류:** `all`, `calendar`, `contacts`, `location`, `location-always`, `photos`, `photos-add`, `media-library`, `microphone`, `camera`, `reminders`, `siri`

---

## 5. swift - Swift Package Manager

### 패키지 생성
```bash
# 라이브러리 패키지 생성
swift package init --type library

# 실행 가능 패키지 생성
swift package init --type executable

# 빈 패키지 생성
swift package init --type empty
```

### 빌드
```bash
# Debug 빌드
swift build

# Release 빌드
swift build -c release

# 특정 타겟만 빌드
swift build --target <TARGET_NAME>

# 상세 출력
swift build -v
```

### 테스트
```bash
# 모든 테스트 실행
swift test

# 특정 테스트만 실행
swift test --filter <TEST_NAME>

# 병렬 테스트
swift test --parallel

# 코드 커버리지
swift test --enable-code-coverage
```

### 실행
```bash
# 기본 실행
swift run

# 특정 타겟 실행
swift run <EXECUTABLE_NAME>

# 인자 전달
swift run <EXECUTABLE_NAME> -- arg1 arg2
```

### 의존성 관리
```bash
# 의존성 해결
swift package resolve

# 의존성 업데이트
swift package update

# 특정 패키지만 업데이트
swift package update <PACKAGE_NAME>

# 의존성 트리 확인
swift package show-dependencies
swift package show-dependencies --format json
```

### 정리
```bash
# 빌드 캐시 정리
swift package clean

# 완전 초기화
swift package reset
```

### Xcode 연동
```bash
# Xcode에서 직접 열기 (권장)
xed .
```

---

## 6. xed - Xcode 에디터 열기

```bash
# 현재 디렉토리 프로젝트 열기
xed .

# 특정 파일 열기
xed <FILE_PATH>

# 특정 줄로 이동하며 열기
xed --line <LINE_NUMBER> <FILE_PATH>

# 백그라운드에서 열기
xed --background <FILE_PATH>
```

---

## 7. agvtool - 버전 관리

### 버전 확인
```bash
# 빌드 번호 확인
agvtool what-version

# 마케팅 버전 확인
agvtool what-marketing-version
```

### 버전 설정
```bash
# 빌드 번호 설정
agvtool new-version -all <BUILD_NUMBER>

# 빌드 번호 자동 증가
agvtool next-version -all

# 마케팅 버전 설정
agvtool new-marketing-version <VERSION>
```

---

## 8. codesign - 코드 서명

### 서명 확인
```bash
# 서명 정보 확인
codesign -dv --verbose=4 <APP_PATH>

# 서명 검증
codesign --verify --deep --strict <APP_PATH>

# 인증서 목록 확인
security find-identity -v -p codesigning
```

### 앱 서명
```bash
# 서명
codesign --sign "<SIGNING_IDENTITY>" \
  --options runtime \
  --timestamp \
  <APP_PATH>

# 강제 재서명
codesign --force --sign "<SIGNING_IDENTITY>" \
  --options runtime \
  --timestamp \
  <APP_PATH>

# 서명 제거
codesign --remove-signature <APP_PATH>
```

---

## 9. notarytool - 앱 공증

### 공증 제출
```bash
# 앱 공증 요청
xcrun notarytool submit <APP_ZIP_OR_DMG> \
  --apple-id "<APPLE_ID>" \
  --password "<APP_SPECIFIC_PASSWORD>" \
  --team-id "<TEAM_ID>" \
  --wait

# Keychain 사용
xcrun notarytool submit <APP_ZIP_OR_DMG> \
  --keychain-profile "<PROFILE_NAME>" \
  --wait
```

### Keychain 프로필 저장
```bash
xcrun notarytool store-credentials "<PROFILE_NAME>" \
  --apple-id "<APPLE_ID>" \
  --password "<APP_SPECIFIC_PASSWORD>" \
  --team-id "<TEAM_ID>"
```

### 공증 상태 확인
```bash
# 상태 확인
xcrun notarytool info <SUBMISSION_ID> \
  --keychain-profile "<PROFILE_NAME>"

# 로그 확인
xcrun notarytool log <SUBMISSION_ID> \
  --keychain-profile "<PROFILE_NAME>"

# 공증 이력 조회
xcrun notarytool history \
  --keychain-profile "<PROFILE_NAME>"
```

### 티켓 스테이플링
```bash
xcrun stapler staple <APP_PATH>
xcrun stapler validate <APP_PATH>
```

---

## 10. altool - App Store Connect 업로드

```bash
# 앱 업로드
xcrun altool --upload-app \
  -f <IPA_OR_PKG_PATH> \
  -t <PLATFORM> \
  -u "<APPLE_ID>" \
  -p "@keychain:AC_PASSWORD"

# 앱 검증만
xcrun altool --validate-app \
  -f <IPA_OR_PKG_PATH> \
  -t <PLATFORM> \
  -u "<APPLE_ID>" \
  -p "@keychain:AC_PASSWORD"
```

**플랫폼:** `ios`, `osx`, `appletvos`

---

## 11. plutil / PlistBuddy - Plist 편집

### plutil
```bash
# Plist 읽기
plutil -p <PLIST_PATH>

# JSON으로 변환
plutil -convert json -o output.json <PLIST_PATH>

# XML로 변환
plutil -convert xml1 <PLIST_PATH>

# 값 삽입
plutil -insert <KEY> -string "<VALUE>" <PLIST_PATH>

# 값 교체
plutil -replace <KEY> -string "<VALUE>" <PLIST_PATH>

# 키 제거
plutil -remove <KEY> <PLIST_PATH>
```

### PlistBuddy
```bash
# 값 읽기
/usr/libexec/PlistBuddy -c "Print :<KEY>" <PLIST_PATH>

# 값 설정
/usr/libexec/PlistBuddy -c "Set :<KEY> <VALUE>" <PLIST_PATH>

# 값 추가
/usr/libexec/PlistBuddy -c "Add :<KEY> <TYPE> <VALUE>" <PLIST_PATH>

# 여러 명령 실행
/usr/libexec/PlistBuddy \
  -c "Set :CFBundleVersion 42" \
  -c "Set :CFBundleShortVersionString 1.0.0" \
  <PLIST_PATH>
```

---

## 12. 자동화 스크립트 템플릿

### 앱 전체 빌드/배포 스크립트
```bash
#!/bin/bash
set -e

# 설정
APP_NAME="MyApp"
SCHEME="MyApp"
CONFIGURATION="Release"
EXPORT_OPTIONS="ExportOptions.plist"

BUILD_DIR="./build"
ARCHIVE_PATH="$BUILD_DIR/$APP_NAME.xcarchive"
EXPORT_PATH="$BUILD_DIR/export"

# 클린
echo "==> Cleaning..."
xcodebuild clean -scheme "$SCHEME" -configuration "$CONFIGURATION"

# 테스트
echo "==> Running tests..."
xcodebuild test \
  -scheme "$SCHEME" \
  -destination 'platform=macOS' \
  -resultBundlePath "$BUILD_DIR/TestResults.xcresult"

# 아카이브
echo "==> Archiving..."
xcodebuild archive \
  -scheme "$SCHEME" \
  -destination 'platform=macOS' \
  -configuration "$CONFIGURATION" \
  -archivePath "$ARCHIVE_PATH"

# 내보내기
echo "==> Exporting..."
xcodebuild -exportArchive \
  -archivePath "$ARCHIVE_PATH" \
  -exportPath "$EXPORT_PATH" \
  -exportOptionsPlist "$EXPORT_OPTIONS"

# 공증 (옵션)
if [ -n "$NOTARIZE" ]; then
  echo "==> Notarizing..."
  ditto -c -k --keepParent "$EXPORT_PATH/$APP_NAME.app" "$BUILD_DIR/$APP_NAME.zip"
  xcrun notarytool submit "$BUILD_DIR/$APP_NAME.zip" \
    --keychain-profile "notarization" \
    --wait
  xcrun stapler staple "$EXPORT_PATH/$APP_NAME.app"
fi

echo "==> Build completed: $EXPORT_PATH"
```

---

## ExportOptions.plist 템플릿

### macOS Developer ID 배포
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>developer-id</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>signingStyle</key>
    <string>automatic</string>
    <key>destination</key>
    <string>export</string>
</dict>
</plist>
```

### iOS App Store 배포
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>signingStyle</key>
    <string>automatic</string>
    <key>uploadBitcode</key>
    <false/>
    <key>uploadSymbols</key>
    <true/>
</dict>
</plist>
```

---

## 참고 자료

- [xcodebuild Man Page](x-man-page://xcodebuild)
- [simctl Man Page](x-man-page://simctl)
- [Xcode Command Line Tools](https://developer.apple.com/library/archive/technotes/tn2339/_index.html)
- [Notarizing macOS Software](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution)
