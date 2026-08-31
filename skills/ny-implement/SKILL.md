---
name: ny-implement
---

# ny-implement (per-phase)

Read `.opencode/context.json`. Find the phase matching the index passed in the prompt. Work through the cycle below for this phase only.

### 1. TDD cycle (tracer bullets)

Load the `/tdd` skill. Drive one behaviour at a time — one failing test, minimal implementation, commit. Repeat until every behaviour in the phase has a test and all tests pass.

After each RED→GREEN step, run `npm run typecheck` and the single test file — catch compilation errors and test failures immediately while the code is fresh.

**Completion**: all phase behaviours have at least one test; `npm test` passes.

### 2. Quality gates

Run these inline (no subagent spawns):

1. **Lint** — `npm run lint`. Zero tolerance. Fix failures and re-run until clean. TDD already covered typecheck + test; lint is the only unchecked dimension.
2. **Audit** — standards review of the phase's changes: does the code follow project conventions and avoid common code smells? Fix each issue, then run `npm run typecheck` after each fix. Repeat until clean.
3. **Coverage** — hunt for gaps from the audit. Add tests or document the decision to skip. Run `npm test` after adding tests. Repeat until clean.
4. **Full verify** — `npm run typecheck && npm test`. Fix regressions from audit/coverage fixes and re-run until all pass. (No lint — already clean from step 1.)

### 3. Commit and push the phase

```bash
if ! git diff --quiet; then
  git add -A && git commit -m "<phase-name>: <summary of what was done>"
fi
git push origin "$(git rev-parse --abbrev-ref HEAD)"
```

The commit message should name the phase and briefly describe what changed.

> **Why commit per phase**: each phase is a reviewable slice of work. Committing here, after the quality gates, bundles the phase's implementation + fixes + tests into one logical unit. The push keeps the remote in sync as a safety net — no lost work if interrupted.

### 4. Mark phase done

Set this phase's `done` to `true` in `.opencode/context.json`.
