---
inclusion: always
---

# Documentation & Changelog Standards

## CHANGELOG.md

This project maintains a living `CHANGELOG.md` at the project root. It serves as a record of all meaningful work completed, organized chronologically by commit.

### Format

Each entry follows this structure:

```markdown
## YYYY-MM-DD — Short Summary of Work

- Bullet describing feature, fix, or change
- Another bullet if multiple things were done
- Reference specific files or components when helpful
```

### Best Practices

- **Be specific**: "Add pagination to /users endpoint" is better than "Update API"
- **Use action verbs**: Start bullets with verbs like Add, Fix, Refactor, Remove, Update, Implement
- **Group related changes**: If a single commit touches auth logic across 5 files, summarize it as one coherent bullet rather than listing each file
- **Note breaking changes**: Prefix with "BREAKING:" if the change affects existing behavior
- **Keep it human-readable**: This document is for the team, not a machine. Write like you're telling a colleague what you did today.
- **Newest first**: Always add new entries at the top of the file, below the header

### What to Include

- New features and capabilities
- Bug fixes (mention what was broken)
- Refactors that change architecture or patterns
- Dependency additions or upgrades
- Configuration changes
- Infrastructure or tooling updates

### What to Skip

- Whitespace-only changes
- Trivial typo fixes (unless in user-facing content)
- Auto-generated file updates (lock files, etc.)

## Commit Messages

Follow conventional commit style:

- `feat:` — new feature
- `fix:` — bug fix
- `refactor:` — code change that neither fixes a bug nor adds a feature
- `docs:` — documentation only
- `chore:` — maintenance tasks, tooling, configs
- `style:` — formatting, missing semicolons, etc.

Keep the subject line under 72 characters. Use imperative mood ("add filter" not "added filter").

## General Documentation Practices

- Document _why_, not just _what_. Code shows what; comments and docs explain intent.
- Keep README.md focused on getting started and high-level architecture.
- Use inline comments sparingly — prefer self-documenting code with clear naming.
- When adding a new system or integration, add a brief section to README or a dedicated doc explaining how it works.
