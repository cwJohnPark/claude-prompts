---
name: ios-app-icon-generator
description: 1024x1024 원본에서 iOS/watchOS/macOS 앱 아이콘 자동 생성 가이드
category: apple
tags: [ios, watchos, macos, app-icon, asset-catalog]
---

# iOS/watchOS/macOS 앱 아이콘 생성 스킬

1024x1024 원본 이미지에서 모든 앱 아이콘을 자동 생성하고 Xcode Asset Catalog 형식으로 출력합니다.

## 요구사항

```bash
pip install Pillow
```

## 사용법

```bash
# 모든 플랫폼 (iOS, watchOS, macOS)
python3 generate_icons.py AppIcon.png

# 특정 플랫폼만
python3 generate_icons.py AppIcon.png ios
python3 generate_icons.py AppIcon.png watchos
python3 generate_icons.py AppIcon.png macos
```

## 출력 구조

```
AppIcons/
├── iOS/AppIcon.appiconset/
│   ├── Contents.json
│   └── icon-*.png
├── watchOS/AppIcon.appiconset/
│   ├── Contents.json
│   └── icon-*.png
└── macOS/AppIcon.appiconset/
    ├── Contents.json
    └── icon-*.png
```

## Xcode에 적용

```bash
# 생성된 폴더를 Assets.xcassets로 복사
cp -r AppIcons/iOS/AppIcon.appiconset "MyApp/Assets.xcassets/"
cp -r AppIcons/watchOS/AppIcon.appiconset "MyWatchApp/Assets.xcassets/"
```

## 아이콘 사이즈

### iOS
| 사이즈 | 용도 |
|--------|------|
| 1024x1024 | App Store |
| 180x180 | iPhone @3x |
| 120x120 | iPhone @2x |
| 167x167 | iPad Pro @2x |
| 152x152 | iPad @2x |
| 76x76 | iPad @1x |

### watchOS
| 사이즈 | 용도 |
|--------|------|
| 1024x1024 | App Store |
| 216x216 | Watch Ultra @2x |
| 196x196 | Watch 45mm @2x |
| 188x188 | Watch 44mm @2x |
| 184x184 | Watch 40mm @2x |
| 176x176 | Watch 38mm @2x |
| 172x172 | Watch 41mm @2x |
| 102x102 | Notification Ultra |
| 88x88 | Notification 42mm |
| 80x80 | Notification 38mm |

### macOS
| 사이즈 | 용도 |
|--------|------|
| 1024x1024 | 512pt @2x |
| 512x512 | 512pt @1x |
| 256x256 | 256pt / 128pt @2x |
| 128x128 | 128pt @1x |
| 64x64 | 32pt @2x |
| 32x32 | 32pt / 16pt @2x |
| 16x16 | 16pt @1x |

## 디자인 가이드라인

- 단순하고 인식하기 쉬운 디자인
- 정사각형 배경 (투명 X)
- 모서리는 시스템이 자동 처리
- 텍스트 포함 지양
- 직접 모서리 둥글게 처리 금지
