---
name: swiftui-git-workflow
description: SwiftUI 프로젝트 Git 커밋 규칙, 브랜치 전략, PR 가이드
category: apple
tags: [swiftui, ios, git, workflow, commit-convention]
---

# SwiftUI Git 워크플로우 스킬

## 개요
SwiftUI 앱 개발 프로젝트의 Git 커밋 메시지 규칙, 브랜치 전략, 협업 워크플로우를 정의합니다.

---

## 커밋 메시지 규칙

### 형식
```
<타입>(<범위>): <제목>

[본문 - 선택사항]

[푸터 - 선택사항]
```

### 타입 (Type)

| 타입 | 설명 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 추가 | `feat: 사용자 프로필 화면 추가` |
| `fix` | 버그 수정 | `fix: 로그인 시 크래시 수정` |
| `refactor` | 코드 리팩토링 (기능 변경 없음) | `refactor: UserViewModel 의존성 주입 개선` |
| `style` | 코드 포맷팅, 세미콜론 누락 등 | `style: 코드 들여쓰기 정리` |
| `docs` | 문서 수정 | `docs: README 설치 가이드 업데이트` |
| `test` | 테스트 추가/수정 | `test: HomeViewModel 유닛 테스트 추가` |
| `chore` | 빌드, 설정, 패키지 매니저 변경 | `chore: SwiftLint 규칙 업데이트` |
| `perf` | 성능 개선 | `perf: 이미지 캐싱 로직 최적화` |
| `ci` | CI/CD 설정 변경 | `ci: GitHub Actions 워크플로우 추가` |
| `revert` | 이전 커밋 되돌리기 | `revert: feat: 사용자 프로필 화면 추가` |

### 범위 (Scope) - 선택사항

변경된 모듈이나 기능 영역을 명시합니다.

```
feat(auth): 소셜 로그인 기능 추가
fix(home): 목록 새로고침 오류 수정
refactor(network): API 클라이언트 구조 개선
```

### 제목 (Subject) 규칙
- **한글**로 작성
- 50자 이내
- 마침표 사용하지 않음
- 명령형으로 작성 (예: "추가", "수정", "개선")

### 본문 (Body) 규칙
- 선택사항
- **한글**로 작성
- 72자마다 줄바꿈
- **무엇**을, **왜** 변경했는지 설명
- **어떻게** 변경했는지는 코드로 확인 가능

### 푸터 (Footer) 규칙
- 선택사항
- 이슈 트래커 ID 참조
- Breaking Change 명시

```
feat(api): 사용자 API 응답 형식 변경

기존 배열 형식에서 페이지네이션을 지원하는 객체 형식으로 변경합니다.
이를 통해 대량의 사용자 목록을 효율적으로 처리할 수 있습니다.

BREAKING CHANGE: fetchUsers API 응답이 [User]에서 PagedResponse<User>로 변경됨
Closes #123
```

---

## 커밋 메시지 예시

### 좋은 예시

```
feat(home): 사용자 검색 기능 추가

- 이름 및 이메일로 검색 가능
- 실시간 필터링 지원
- 검색 결과 하이라이팅

Closes #45
```

```
fix(auth): 토큰 만료 시 자동 갱신 안 되는 문제 수정

refreshToken이 nil인 경우를 처리하지 않아 발생한 문제입니다.
옵셔널 바인딩 추가로 해결했습니다.

Fixes #78
```

```
refactor(viewmodel): MVVM 패턴 적용

- View에서 비즈니스 로직 분리
- ViewModel에 상태 관리 집중
- 테스트 용이성 향상
```

### 나쁜 예시

```
수정함
```

```
버그 fix
```

```
feat: Added user search feature and fixed some bugs and also refactored the code
```

---

## 브랜치 전략

### Git Flow

```
main (또는 master)
  │
  ├── develop
  │     │
  │     ├── feature/사용자-검색
  │     ├── feature/프로필-수정
  │     └── feature/알림-기능
  │
  ├── release/1.0.0
  │
  └── hotfix/로그인-크래시
```

### 브랜치 명명 규칙

| 브랜치 타입 | 형식 | 예시 |
|------------|------|------|
| 기능 개발 | `feature/<기능명>` | `feature/user-search` |
| 버그 수정 | `bugfix/<이슈번호>-<설명>` | `bugfix/123-login-crash` |
| 긴급 수정 | `hotfix/<버전>-<설명>` | `hotfix/1.0.1-token-refresh` |
| 릴리스 | `release/<버전>` | `release/1.0.0` |

### 브랜치 작업 흐름

```bash
# 1. develop에서 feature 브랜치 생성
git checkout develop
git pull origin develop
git checkout -b feature/user-search

# 2. 작업 후 커밋
git add .
git commit -m "feat(search): 사용자 검색 UI 구현"

# 3. 원격에 푸시
git push origin feature/user-search

# 4. Pull Request 생성 -> 코드 리뷰 -> Merge
```

---

## Pull Request 가이드

### PR 템플릿

```markdown
## 개요
<!-- 이 PR에서 변경한 내용을 간략히 설명해주세요 -->

## 관련 이슈
<!-- 관련 이슈 번호를 링크해주세요 -->
Closes #

## 체크리스트
- [ ] 코드가 프로젝트 스타일 가이드를 따르나요?
- [ ] 자체 코드 리뷰를 수행했나요?
- [ ] 주석을 추가했나요? (특히 이해하기 어려운 부분)
- [ ] 문서를 업데이트했나요?
- [ ] 변경 사항이 새로운 경고를 생성하지 않나요?
- [ ] 테스트를 추가했나요?
- [ ] 모든 테스트가 통과하나요?

## 스크린샷 (UI 변경 시)
<!-- UI 변경이 있다면 스크린샷을 첨부해주세요 -->

## 리뷰어에게
<!-- 리뷰어가 특별히 확인해야 할 부분이 있다면 적어주세요 -->
```

### PR 규칙
- **작은 단위**로 PR 생성 (300줄 이하 권장)
- **한 가지 목적**만 달성하는 PR
- 리뷰어 **최소 1명** 승인 필요
- 모든 **테스트 통과** 필수
- **충돌 해결** 후 Merge

---

## 코드 리뷰 가이드

### 리뷰어 체크리스트

```markdown
## 기능
- [ ] 요구사항을 충족하는가?
- [ ] 엣지 케이스를 처리하는가?

## 코드 품질
- [ ] 코딩 규칙을 따르는가?
- [ ] 불필요한 복잡성이 없는가?
- [ ] 적절한 추상화 수준인가?

## 성능
- [ ] 성능 저하 가능성이 있는가?
- [ ] 메모리 누수 가능성이 있는가?

## 테스트
- [ ] 테스트 커버리지가 충분한가?
- [ ] 테스트가 의미 있는가?

## 문서
- [ ] 주석이 적절한가?
- [ ] README 업데이트가 필요한가?
```

---

## 버전 관리

### Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH

예: 1.2.3
```

| 버전 | 변경 시점 |
|------|----------|
| **MAJOR** | 호환되지 않는 API 변경 |
| **MINOR** | 하위 호환되는 기능 추가 |
| **PATCH** | 하위 호환되는 버그 수정 |

### 태그 생성

```bash
# 릴리스 태그 생성
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# 태그 목록 확인
git tag -l
```

---

## .gitignore 설정

```gitignore
# Xcode
*.xcuserdata/
*.xcscmblueprint
*.xccheckout
DerivedData/
*.moved-aside
*.pbxuser
*.mode1v3
*.mode2v3
*.perspectivev3
xcuserdata/
*.xcworkspace
!default.xcworkspace

# Swift Package Manager
.build/
Packages/
Package.pins
Package.resolved

# CocoaPods (사용 시)
Pods/

# Carthage (사용 시)
Carthage/Build/

# fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots/**/*.png
fastlane/test_output

# macOS
.DS_Store
.AppleDouble
.LSOverride
._*

# IDE
*.swp
*.swo
.idea/

# Environment
.env
.env.local

# Build
build/
*.ipa
*.dSYM.zip
*.dSYM
```

---

## 유용한 Git 별칭

```bash
# ~/.gitconfig에 추가
[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    lg = log --oneline --graph --decorate --all
    unstage = reset HEAD --
    last = log -1 HEAD

    # 브랜치 정리
    cleanup = "!git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d"

    # 대화형 리베이스
    ri = rebase -i
```

---

## 참고 자료

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [Semantic Versioning](https://semver.org/)
