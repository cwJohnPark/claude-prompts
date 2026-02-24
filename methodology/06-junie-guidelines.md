---
name: junie-guidelines
description: JetBrains Junie AI 에이전트 프로젝트 가이드라인 - SOLID, 코딩 스타일, 테스트, 커밋 규칙
category: methodology
tags: [methodology, junie, jetbrains, guidelines, solid, coding-style]
---

# Project Guidelines

### General Rules

- Write less than 100 lines per file when possible.
- Keep methods small and cohesive.
- Avoid nested logic depth; prefer flat structures with clear exits.
- Apply SOLID principles consistently.
- Separate concerns: controller, service, repository, config, domain.
- Prefer immutable data classes.

### SOLID Application

- **Single Responsibility**: Each class or method does one thing only.
- **Open/Closed**: Extend behavior via interfaces and implementations, never modify existing stable code.
- **Liskov Substitution**: Replace abstractions with implementations safely.
- **Interface Segregation**: Use focused interfaces instead of large general contracts.
- **Dependency Inversion**: Depend on abstractions, inject via framework DI.

### Comments

- Write comments in Korean for better readability by the team.
- Add comments for complex logic or non-obvious implementation details.

## Agent Instructions

- When defining classes, mark full-argument constructors as `private` and expose static factory methods (or builders) for instantiation.
- Document every controller endpoint with API documentation annotations, keeping them in dedicated annotation interfaces colocated with their controllers. All textual elements must be written in Korean.
- All REST controller endpoints must live under the `/api/v1` path prefix.
- Prefer Streams over traditional `for` loops in all new code.

### Browser Automation & Testing

- Use Playwright MCP when frameworks or libraries are required.
- Playwright MCP provides browser automation capabilities through Model Context Protocol.
- Available tools include navigation, form filling, screenshots, and accessibility tree parsing.

## Coding Style & Naming Conventions

- 4-space indentation; keep imports organized.
- Classes use `PascalCase`, components/services end with `Controller`, `Service`, `Repository`.
- Use Lombok only where it reduces boilerplate; favor explicit constructors for required dependencies.

## Testing Guidelines

- Tests use JUnit 5 with framework test starters; integration suites leverage in-memory DB.
- Name test classes `<Subject>Test` and individual methods `should<Expectation>()`.
- Strive for coverage on business rules; add regression tests when touching critical flows.

## Commit & Pull Request Guidelines

- Follow Conventional Commits (`feat(api): add user endpoint`, `ci: adjust pipeline cache`), using lowercase type and optional module scope.
- Keep commits focused and run tests before pushing; include config or migrations in the same change.
- PRs need a short summary, verification notes, linked issue, and updated docs if behavior shifts.

## Important Rules

- Never execute `git reset --hard` or `git restore`
- Never revert files that the user has manually modified
