---
name: review
description: Perform a comprehensive code review covering PHP, Symfony, Laravel, and TypeScript/React patterns. Use when the user says "review this code", "code review", "check this PR", "review my changes", or wants a structured quality assessment.
---

# Code Review

Perform comprehensive code review. Be thorough but concise.

## Check For

### PHP (all backend repos)

- **Debug statements** -- No `var_dump()`, `dd()`, `dump()`, `print_r()`, `error_log()` left in code
- **Type safety** -- Proper type hints on parameters, return types, and properties. No mixed types without justification
- **PSR-12** -- Coding style compliance
- **Error handling** -- Try-catch for external calls, meaningful exception messages, no silent failures
- **SQL / ORM** -- No raw queries without parameter binding, no N+1 queries, proper use of Doctrine/Eloquent
- **Security** -- No SQL injection vectors, no XSS in templates, CSRF protection on forms, input validation
- **Architecture** -- Follows existing module/service patterns, code in correct directory under `source/src/`
- **Tests** -- PHPUnit tests for new logic, fixtures for data-dependent tests

### Symfony-specific (payments-gateway, payments-settings)

- **Services** -- Proper dependency injection, no `new` in controllers
- **Doctrine** -- Migrations included for schema changes, entities have proper annotations/attributes
- **Temporal** -- Workflow activities are idempotent (payments-gateway)

### Laravel-specific (transfergo monolith)

- **Eloquent** -- Eager loading where needed, no lazy loading in loops
- **Blade** -- `{{ }}` for escaped output, `{!! !!}` only when explicitly safe
- **Routes** -- Middleware applied, proper naming

### TypeScript / React (admin-frontend-old)

- **Types** -- No `any` types, proper interfaces
- **Hooks** -- Effects have cleanup, dependency arrays complete
- **Performance** -- No unnecessary re-renders, expensive computations memoized
- **Ant Design** -- Follows existing component patterns

### Universal

- **Production readiness** -- No debug code, no TODOs, no hardcoded secrets
- **Logging** -- Uses proper logger with context, not console/echo
- **Secrets** -- No credentials, tokens, or PII in code

## Output Format

### Looks Good
- [Item 1]
- [Item 2]

### Issues Found
- **[Severity]** [File:line] -- [Issue description]
  - Fix: [Suggested fix]

### Summary
- Files reviewed: X
- Critical issues: X
- Warnings: X

## Severity Levels
- **CRITICAL** -- Security vulnerabilities, data loss, crashes
- **HIGH** -- Bugs, performance issues, bad UX
- **MEDIUM** -- Code quality, maintainability
- **LOW** -- Style, minor improvements
