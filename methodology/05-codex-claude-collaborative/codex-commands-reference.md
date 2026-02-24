---
name: codex-commands-reference
description: Codex CLI 명령어 템플릿 및 사용법 참조
category: methodology
tags: [methodology, codex, cli, commands]
---

# Codex CLI Commands Reference

이 문서는 프로젝트에서 자주 사용하는 Codex CLI 명령어 템플릿을 제공합니다.

## 기본 명령어 구조

```bash
codex exec "[prompt]" \
  --model gpt-5-codex \
  --sandbox workspace-write \
  -C /path/to/project \
  --ask-for-approval on-failure
```

## Command Templates by Use Case

### 1. Value Object 생성

```bash
codex exec "Create a Value Object class named [ClassName] with these requirements:

REQUIREMENTS:
- Package: com.example.core.domain.[subdomain]
- Immutable design (all fields final)
- Field: [fieldType] [fieldName] - [description]
- Private constructor with validation
- Static factory method: [ClassName].of([params])
- Validation rules: [rules]
- Implement equals/hashCode
- Korean JavaDoc comments
- NO else clauses (use early return)
- Maximum 50 lines

OUTPUT:
- Full class implementation
- Include package declaration and imports
"
```

### 2. Entity 생성

```bash
codex exec "Create an Entity class named [EntityName] with these requirements:

REQUIREMENTS:
- Identity field: Long id
- Business fields: [list]
- Immutable Value Objects for domain concepts
- Business methods (not getters/setters)
- Private constructor
- Static factory method: [EntityName].create([params])
- NO else clauses
- Maximum 50 lines per class

OUTPUT:
- Entity class implementation
"
```

### 3. First-Class Collection

```bash
codex exec "Create a First-Class Collection named [CollectionName]:

REQUIREMENTS:
- Wraps: List<[ElementType]>
- Immutable (unmodifiable list)
- Private constructor
- Static factory: [CollectionName].of(List<[ElementType]>)
- Business methods: [list]
- Validation in constructor
- NO else clauses
- Maximum 50 lines

OUTPUT:
- Full implementation
"
```

### 4. Service Layer Method

```bash
codex exec "Implement a service method:

REQUIREMENTS:
- Method signature: public [ReturnType] [methodName]([params])
- Business logic: [description]
- Error handling: [rules]
- NO else clauses
- Maximum 20 lines per method

OUTPUT:
- Method implementation only
"
```

### 5. Test Cases Generation

```bash
codex exec "Generate comprehensive test cases for [ClassName]:

REQUIREMENTS:
- Framework: JUnit 5 + AssertJ
- Structure: Given-When-Then
- @DisplayName in Korean
- Test categories: happy path, boundary, exception, edge cases
- Coverage: 100% of public methods
- Each test method <=20 lines

OUTPUT:
- Full test class implementation
"
```

## Session Management

```bash
# Resume last session
codex resume --last

# Clear session
codex clear
```

## Safety Flags

| Flag | Description | When to Use |
|------|-------------|-------------|
| `--sandbox workspace-write` | Limit to workspace | **Always** (default) |
| `--sandbox read-only` | Read files only | Exploration phase |
| `--ask-for-approval on-failure` | Confirm on errors | **Always** (recommended) |
| `--ask-for-approval always` | Confirm every action | Critical code |
| `--danger-full-access` | No restrictions | **Never** use |

## Tips for Better Results

### 1. Be Specific About Constraints

```bash
# Bad
codex exec "Create a value object for distance"

# Good
codex exec "Create Distance Value Object:
- double meters field
- Validation: meters >= 0
- Conversions: toKilometers()
- Korean JavaDoc
- <=50 lines
- NO else clauses"
```

### 2. Provide Context

Include project type, language version, existing patterns

### 3. Specify Output Format

Request pure code, no markdown, compilable as-is

## Quick Reference Card

| Need | Command Template |
|------|------------------|
| **Value Object** | `codex exec "Create [Name] VO: immutable, validation, <=50 lines"` |
| **Entity** | `codex exec "Create [Name] Entity: business methods, DDD"` |
| **First-Class Collection** | `codex exec "Create [Name]s wrapping List<[Type]>, business methods"` |
| **Service Method** | `codex exec "Implement [method] in [Service]: [logic]"` |
| **Tests** | `codex exec "Generate tests for [Class]: Given-When-Then"` |
