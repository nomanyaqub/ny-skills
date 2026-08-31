---
name: ny-audit
description: "Audit the implemented changes against the plan: missed concerns, incorrect assumptions, edge cases, correctness and consistency with existing code, project conventions, and security. Use when a code audit is requested after implementation."
---

# ny-audit

Read the changes as an **adversary** looking for flaws. Examine the current phase's changes:

- Missed concerns, incorrect assumptions, and edge cases
- Correctness and consistency with existing code
- Compliance with project conventions (from `AGENTS.md` and `docs/`)
- Security concerns

For each item, document a finding — either clean or flagged with the issue. Do not skip items.

**Completion**: every check item examined; each has a documented finding (clean or flagged).
