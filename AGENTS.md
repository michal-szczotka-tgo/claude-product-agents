# AGENTS.md

## Cursor Cloud specific instructions

This is a **documentation-only repository** — there are no source code files, no dependencies, no build system, no tests, and no services to run.

### What this repo contains

19 Claude Code slash command files (`commands/*.md`) covering the full product-to-engineering pipeline. See `README.md` for the pipeline overview and `agents.md` (lowercase) for the full command reference.

### "Installation" / validation

The only meaningful action is copying command files to `~/.claude/commands/`:

```bash
mkdir -p ~/.claude/commands && cp commands/*.md ~/.claude/commands/
```

### Linting / testing / building

- **No linter** is configured. Validation is limited to checking that each `.md` file in `commands/` has valid YAML frontmatter (`---` delimiters, `name`, `description` fields).
- **No automated tests** exist. The commands are validated by manual usage in Claude Code sessions.
- **No build step** exists. The markdown files are consumed directly by Claude Code.

### Contributing

When adding or editing commands in `commands/`, follow the existing format: YAML frontmatter with `name` and `description`, followed by numbered steps with clear trigger conditions. See `README.md` § Contributing.
