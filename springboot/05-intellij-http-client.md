---
name: intellij-http-client
description: IntelliJ HTTP Client 파일 작성 가이드. 변수를 public/private으로 분리하여 민감 정보가 Git에 추적되지 않도록 관리
category: springboot
tags: [spring-boot, java, intellij, http-client, testing]
---

# IntelliJ HTTP Client Guide

이 스킬은 IntelliJ HTTP Client 파일 작성 및 변수 관리 가이드라인을 제공합니다.

## When to Use This Skill

### 자동 트리거

- `.http` 파일 작성/수정 시
- `http-client.env.json` 또는 `http-client.private.env.json` 파일 작성 시
- HTTP 요청 테스트 파일 생성 시

---

## 핵심 원칙

**Private 변수는 자주 변경되므로 Git에서 추적하지 않는다.**

| 구분 | 파일 | Git 추적 | 용도 |
|------|------|----------|------|
| Public | `http-client.env.json` | 추적됨 | 환경별 baseUrl |
| Private | `http-client.private.env.json` | .gitignore | 모든 민감/가변 데이터 |

---

## 파일 구조

각 모듈의 `http/` 디렉토리:

```
module/http/
├── http-client.env.json           # Public (Git 추적)
├── http-client.private.env.json   # Private (Git 제외)
├── api/
│   └── *.http
└── *.http
```

---

## 변수 분류

### Public Variables (`http-client.env.json`)

환경별 기본 URL만 포함:

```json
{
  "dev": {
    "baseUrl": "http://localhost:8080"
  },
  "staging": {
    "baseUrl": "https://staging-api.example.com"
  },
  "prod": {
    "baseUrl": "https://api.example.com"
  }
}
```

### Private Variables (`http-client.private.env.json`)

나머지 모든 변수:

```json
{
  "dev": {
    "baseUrl": "http://localhost:7020",

    "token": "eyJhbGciOiJIUzI1NiIs...",
    "API_KEY": "sk-1234567890",

    "resourceId": "123",
    "code": "PRJ-001",
    "fromDateTime": "2025-01-01T00:00:00",
    "toDateTime": "2025-03-01T00:00:00",

    "EXTERNAL_API_USERNAME": "api-user",
    "EXTERNAL_API_PASSWORD": "secret123"
  }
}
```

**Private에 포함되는 항목:**
- `token` - 인증 토큰
- `API_KEY`, `*_PASSWORD`, `*_SECRET` - 인증 정보
- Request Parameters - `resourceId`, `code` 등
- Request Body Values - `fromDateTime`, `toDateTime` 등
- 외부 API 인증 정보

---

## 네이밍 컨벤션

### 변수 참조

```http
# 올바른 형식 (공백 없음)
GET {{baseUrl}}/api/v1/resources/{{resourceId}}
Authorization: Bearer {{token}}

# 잘못된 형식 (공백 있음)
GET {{ baseUrl }}/api/v1/resources/{{ resourceId }}
```

### 변수명 규칙

| 타입 | 네이밍 | 예시 |
|------|--------|------|
| 일반 변수 | camelCase | `baseUrl`, `resourceId`, `fromDateTime` |
| 인증/비밀 | SCREAMING_SNAKE_CASE | `API_KEY`, `SERVICE_PASSWORD` |
| 외부 서비스 | PREFIX_SNAKE_CASE | `AWS_ACCESS_KEY`, `EXTERNAL_USERNAME` |

---

## HTTP 파일 템플릿

### 기본 API 요청

```http
### API 설명
GET {{baseUrl}}/api/v1/resource/{{resourceId}}
Authorization: Bearer {{token}}
Content-Type: application/json
```

### POST 요청 (Body 포함)

```http
### 리소스 생성
POST {{baseUrl}}/api/v1/resources
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "{{resourceName}}",
  "startDate": "{{fromDateTime}}",
  "endDate": "{{toDateTime}}"
}
```

### 외부 API 호출

```http
@externalApiUrl = https://external-api.example.com

### 외부 API 인증
POST {{externalApiUrl}}/auth/token
Content-Type: application/json

{
  "username": "{{EXTERNAL_USERNAME}}",
  "password": "{{EXTERNAL_PASSWORD}}"
}
```

---

## 실제 예시

### 예시 모듈

**http-client.env.json** (Public):
```json
{
  "dev": {
    "baseUrl": "http://localhost:8080"
  }
}
```

**http-client.private.env.json** (Private):
```json
{
  "dev": {
    "baseUrl": "http://localhost:8080",
    "resourceId": "123",
    "fromDateTime": "2025-01-01T00:00:00",
    "toDateTime": "2025-03-01T00:00:00",
    "EXTERNAL_API_KEY": "sk-1234...",
    "EXTERNAL_USERNAME": "api-user",
    "EXTERNAL_PASSWORD": "secret"
  }
}
```

**api/batch.http**:
```http
### RUN Batch Process (default)
POST {{baseUrl}}/api/v1/batch/process
Content-Type: application/json

### RUN Batch Process (with parameters)
POST {{baseUrl}}/api/v1/batch/process
Content-Type: application/json

{
  "fromDateTime": "{{fromDateTime}}",
  "toDateTime": "{{toDateTime}}",
  "resourceId": "{{resourceId}}"
}
```

---

## .gitignore 설정

프로젝트 루트의 `.gitignore`에 다음이 포함되어야 함:

```gitignore
# IntelliJ HTTP Client private environment
http-client.private.env.json
```

---

## 체크리스트

HTTP 파일 작성 시 확인사항:

- [ ] `baseUrl`만 `http-client.env.json`에 있는가?
- [ ] 나머지 모든 변수는 `http-client.private.env.json`에 있는가?
- [ ] 변수 참조에 공백이 없는가? `{{var}}` (O) `{{ var }}` (X)
- [ ] `http-client.private.env.json`이 `.gitignore`에 포함되어 있는가?
- [ ] 하드코딩된 값이 없는가?

---

## Quick Reference

### 파일 용도

| 파일 | 용도 | Git |
|------|------|-----|
| `http-client.env.json` | 환경별 baseUrl | 추적 |
| `http-client.private.env.json` | 민감/가변 데이터 | 제외 |
| `*.http` | HTTP 요청 정의 | 추적 |

### 변수 분류

| Private 변수 | 예시 |
|--------------|------|
| 인증 토큰 | `token`, `API_KEY` |
| 인증 정보 | `*_USERNAME`, `*_PASSWORD` |
| Request Params | `resourceId`, `code` |
| Request Body | `fromDateTime`, `toDateTime` |
| 외부 API | `AWS_*`, `EXTERNAL_*` |

---

이 스킬을 사용하여 민감 정보가 Git에 노출되지 않도록 하고, 변경이 잦은 테스트 값들이 불필요하게 커밋되지 않도록 관리하세요.
