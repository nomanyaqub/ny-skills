---
name: ny-verify
description: Run lint, typecheck, and tests with zero tolerance, fixing issues and re-running until all pass. Use to verify pipeline changes before review.
---

# ny-verify

Zero-tolerance: any failure blocks advancement, no partial passes.

Run all three checks with zero-tolerance:

1. `npm run lint`
2. `npm run typecheck`
3. `npm test`

If any check fails, fix the issue and re-run all three from the start. All must pass before proceeding.

**Completion**: all three checks pass with zero failures.
