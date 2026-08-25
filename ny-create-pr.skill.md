---
name: ny-create-pr
---

# ny-create-pr

Read `.opencode/context.json` for the issue number, title, and branch name.

1. Create a PR to `main` using `gh pr create`
2. Title: `"Issue <number>: <title>"`
3. Body includes:
   - Acceptance criteria from the issue
   - `Closes #<number>`
4. Leave open for review — do not merge
