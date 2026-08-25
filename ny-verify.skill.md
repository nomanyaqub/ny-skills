---
name: ny-verify
---

# ny-verify

Zero-tolerance: any failure blocks advancement, no partial passes.

Run all three checks with zero-tolerance:

1. `npm run lint`
2. `npm run typecheck`
3. `npm test`

If any check fails, fix the issue and re-run all three from the start. All must pass before proceeding.

**Completion**: all three checks pass with zero failures.
