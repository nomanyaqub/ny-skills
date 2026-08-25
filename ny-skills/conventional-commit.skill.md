---
name: conventional-commit
description: Format all commit messages as conventional commits using commitlint config.
---

# Conventional Commit Format

All commit messages **must** follow the format:

```
type(scope): description
```

- `type` is one of the allowed types (see below)
- `scope` is optional but recommended (e.g., `api`, `ui`, `deps`, `config`)
- `description` is a brief present-tense summary (lowercase, no period)

## Allowed types

| Type       | Usage                              |
| ---------- | ---------------------------------- |
| `feat`     | A new feature                      |
| `fix`      | A bug fix                          |
| `docs`     | Documentation changes              |
| `style`    | Code style (formatting, no logic)  |
| `refactor` | Code refactoring                   |
| `perf`     | Performance improvements           |
| `test`     | Adding or fixing tests             |
| `chore`    | Build, CI, tooling, dependencies   |
| `ci`       | CI configuration changes           |

## Examples

```
feat(api): add upload endpoint
fix(ui): correct button alignment on mobile
docs: update README with new API paths
refactor(auth): extract token validation into middleware
chore(deps): bump zod to 3.24
test(upload): add edge cases for empty files
```

## Enforcement

This project uses `commitlint` with `@commitlint/config-conventional` to enforce the format. The commit-msg git hook (via husky) runs automatically on every `git commit`.
