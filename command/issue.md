---
description: Work through a GitHub issue end-to-end (branch → plan → build → verify → review → PR)
---

You are working on GitHub issue #$1.

1. Run `gh issue view $1 --json title,body,labels,assignees` to verify the issue exists
2. Create `.opencode/context.json` with `{"issueNumber": $1}`
3. Follow the `ny-pipeline` skill from start to finish
