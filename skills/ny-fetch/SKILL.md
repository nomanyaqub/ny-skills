---
name: ny-fetch
---

# ny-fetch

Read `issueNumber` from `.opencode/context.json`. Then:

1. Fetch issue details: `gh issue view <number> --json title,body,labels,assignees`
2. Parse the title into a kebab-case branch name
3. `git fetch origin main && git checkout main && git pull origin main`
4. Create a branch: `issue-<number>-<kebab-case-title>`
5. Update `.opencode/context.json` with the issue title, body, and branch name (preserve existing fields — merge, do not overwrite)
