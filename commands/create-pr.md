---
name: create-pr
description: Create a GitHub pull request end-to-end -- branch creation, commits, push, and structured PR description. Use when the user says "create PR", "open pull request", "submit changes", "push my changes", or is ready to ship code changes to the repository.
---

# Create Pull Request

End-to-end workflow for shipping code changes as a GitHub PR.

## Prerequisites

- You are in a git repository with a remote origin
- `gh` CLI is authenticated (`gh auth status`)
- Changes are already implemented (use the `execute` skill first)

## Workflow

### 1. Branch

```bash
git checkout -b <type>/<short-description>
```

Branch naming: `feat/`, `fix/`, `chore/`, `docs/` prefix + kebab-case description.
If already on a feature branch, skip this step.

### 2. Stage and Commit

- Review all changes with `git diff`
- Stage relevant files only (no debug files, no secrets)
- Write a commit message following conventional commits:

```
<type>(<scope>): <short summary>

<body - what and why, not how>
```

### 3. Push

```bash
git push -u origin HEAD
```

### 4. Create PR

```bash
gh pr create --title "<type>(<scope>): <summary>" --body "$(cat <<'EOF'
## Summary
<2-3 bullet points: what changed and why>

## Changes
- <file/module>: <what changed>

## Testing
- [ ] <how to verify this works>

## Jira
<TICKET-ID if available>
EOF
)"
```

## Quality Checks Before Submitting

- [ ] No console.log / debug statements
- [ ] No hardcoded secrets or credentials
- [ ] No unrelated changes included
- [ ] PR title is clear and follows convention
- [ ] Description explains the WHY, not just the WHAT
