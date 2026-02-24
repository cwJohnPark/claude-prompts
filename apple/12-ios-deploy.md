---
name: ios-deploy
description: iOS/watchOS 앱 App Store Connect 배포 가이드 (Archive, 업로드, 심사)
category: apple
tags: [ios, watchos, deploy, app-store, xcodebuild, archive]
---

# iOS/watchOS App Store 배포 스킬

iOS 및 watchOS 앱을 App Store Connect에 배포합니다.

## 실행 방법

### 전체 배포 (Archive + 업로드)
```bash
./scripts/deploy-appstore.sh
```

### Archive만 생성 (업로드 건너뛰기)
```bash
./scripts/deploy-appstore.sh --skip-upload
```

## 사전 준비

### 1. App-Specific Password 생성
1. https://appleid.apple.com 접속
2. 보안 > 앱 암호 > 암호 생성
3. 생성된 암호 복사 (xxxx-xxxx-xxxx-xxxx 형식)

### 2. Keychain에 암호 저장
```bash
xcrun altool --store-password-in-keychain-item "AC_PASSWORD" \
  -u "your@email.com" \
  -p "xxxx-xxxx-xxxx-xxxx"
```

### 3. 환경변수 설정 (선택)
```bash
export APPLE_ID="your@email.com"
```

## 배포 단계

1. **빌드 번호 자동 증가** (agvtool)
2. **Archive 생성** (xcodebuild archive)
3. **IPA 내보내기** (xcodebuild -exportArchive)
4. **App Store Connect 업로드** (xcrun altool)

## 배포 후 작업

1. [App Store Connect](https://appstoreconnect.apple.com) 접속
2. 앱 선택 > 빌드 확인
3. 새 버전 또는 기존 버전에 빌드 추가
4. 심사 정보 입력
5. "심사를 위해 제출" 클릭

## 파일 구조

```
ProjectName/
├── ExportOptions.plist          # 내보내기 옵션
├── build/
│   ├── ProjectName.xcarchive   # Archive 파일
│   └── *.ipa                    # 내보낸 IPA
└── scripts/
    └── deploy-appstore.sh       # 배포 스크립트
```

## 배포 스크립트 예시

```bash
#!/bin/bash
set -e

SCHEME="MyApp"
CONFIGURATION="Release"
ARCHIVE_PATH="./build/MyApp.xcarchive"
EXPORT_PATH="./build/export"
EXPORT_OPTIONS="ExportOptions.plist"

# 빌드 번호 증가
agvtool next-version -all

# Archive 생성
xcodebuild archive \
  -scheme "$SCHEME" \
  -destination 'generic/platform=iOS' \
  -configuration "$CONFIGURATION" \
  -archivePath "$ARCHIVE_PATH"

# IPA 내보내기
xcodebuild -exportArchive \
  -archivePath "$ARCHIVE_PATH" \
  -exportPath "$EXPORT_PATH" \
  -exportOptionsPlist "$EXPORT_OPTIONS"

# App Store Connect 업로드
if [ "$1" != "--skip-upload" ]; then
  xcrun altool --upload-app \
    -f "$EXPORT_PATH/MyApp.ipa" \
    -t ios \
    -u "${APPLE_ID}" \
    -p "@keychain:AC_PASSWORD"
fi

echo "Done!"
```

## ExportOptions.plist

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

### watchOS 앱 배포

watchOS 앱은 iOS 앱에 포함되거나 독립 앱으로 배포합니다:

```bash
# watchOS companion 앱 (iOS 앱과 함께)
xcodebuild archive \
  -scheme "MyApp" \
  -destination 'generic/platform=iOS' \
  -archivePath "./build/MyApp.xcarchive"

# watchOS 독립 앱
xcodebuild archive \
  -scheme "MyWatchApp" \
  -destination 'generic/platform=watchOS' \
  -archivePath "./build/MyWatchApp.xcarchive"
```

## 문제 해결

### Archive 실패
- 인증서/프로비저닝 프로파일 확인
- Xcode > Preferences > Accounts에서 팀 로그인 확인

### 업로드 실패
- App-Specific Password 재생성
- Bundle ID가 App Store Connect에 등록되어 있는지 확인
- 빌드 번호가 이전보다 높은지 확인

### watchOS 관련
- watchOS 앱의 Bundle ID는 iOS 앱의 하위여야 함 (예: `com.app.myapp.watchkitapp`)
- 독립 watchOS 앱은 `WKRunsIndependentlyOfCompanionApp = YES` 설정 필요
