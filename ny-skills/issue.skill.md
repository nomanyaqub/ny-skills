---
name: issue
description: Work through a GitHub issue end-to-end (branch → plan → build → verify → review → PR). Use when the user gives an issue number or asks to work on issue #N.
---

You are working on a GitHub issue. Use the issue number the user provided.

1. Run `gh issue view <number> --json title,body,labels,assignees` to verify the issue exists
2. Create `.opencode/context.json` with `{"issueNumber": <number>}`
3. Follow the `ny-pipeline` skill from start to finish
