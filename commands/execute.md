---
name: execute
description: Implement a feature precisely as planned, writing clean modular code and tracking progress. Use when the user says "execute", "implement the plan", "build it", "start building", "go ahead and implement", or is ready to move from planning to implementation.
---

# Execute Implementation

Implement precisely as planned, in phases.

## Before Writing Code

1. Create a feature branch:

```bash
git checkout -b <type>/<short-description>
```

Branch types: `feat/`, `fix/`, `chore/`, `docs/`, `refactor/`

2. Re-read the repo's `AGENTS.md` or `CLAUDE.md` for coding conventions
3. Confirm the plan with the user if anything is ambiguous

## Implementation Rules

- Write minimal, clean code that follows existing patterns in the repo
- Match the code style of surrounding files exactly
- Use proper type hints (PHP) or types (TypeScript)
- Handle errors explicitly -- no silent failures
- Add tests for new logic (PHPUnit for PHP, Vitest for TS)

## Phase-by-Phase Execution

For each phase of the plan:

1. Implement the changes
2. Run existing tests if a test command is available (`make test`, `php artisan test`, `pnpm test`)
3. Report what changed:
   - Files modified / created / deleted
   - Key decisions made during implementation
   - Any deviations from the plan and why
4. Wait for user confirmation before proceeding to next phase

## After All Phases Complete

1. Run a self-review using the `review` skill checklist
2. Ensure no debug statements, no hardcoded values, no TODOs left
3. Commit with conventional commit messages
4. Update the tracking document with progress if one exists
