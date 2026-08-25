---
name: ny-pipeline
---

# ny-pipeline

Read `.opencode/context.json`. Steps 1–2 (Plan, Implement) run by spawning fresh subagents via the Task tool — **wait for each subagent to complete before spawning the next**, and pass each the context file path so it can read and update state. Steps 3–5 (Lightweight verify, Code review, Address findings) run **in the main pipeline thread** — no delegation — except the two review axes in step 4, which fan out in parallel. Step 6 (Open PR) runs in the main thread. Step 7 (Post-PR external review) runs **in the main thread** and delegates only each round's review itself to the `pr-reviewer` subagent (a read-only agent defined globally in `~/.config/opencode/opencode.jsonc`).

The individual `ny-*` skill files (ny-fetch, ny-plan, ny-verify, ny-audit, ny-coverage, ny-review-docs, ny-document, ny-create-pr) remain as independently callable reference — the pipeline subagents follow them inline rather than spawning sub-subagents.

## Steps

### 1. Plan

Spawn a Task subagent that does the following inline:

1. **Fetch**: Read `issueNumber` from `.opencode/context.json`. `gh issue view <number> --json title,body,labels,assignees`. Parse the title into a kebab-case branch name. `git fetch origin main && git checkout main && git pull origin main`. Create branch `issue-<number>-<kebab-case-title>`. Update `.opencode/context.json` with the issue title, body, and branch name (merge, don't overwrite).
2. **Plan**: Read `.opencode/context.json` and `.opencode/fetched-issue.json`. Study the relevant code. Present a plan covering what files change, approach, decisions, and phase splits. Parse acceptance criteria (`- [ ]` items) and assign to phases.
3. **Gate**: Wait for the user to say "go ahead" (or similar). If rejected, abort.
4. If approved, write phases and acceptance criteria into `.opencode/context.json`.

**Completion**: context.json is populated with phases and ACs, or the pipeline is aborted.

### 2. Implement (per-phase)

Read `phases` from `.opencode/context.json`. For each phase where `done` is `false`, spawn a Task subagent **in sequence** (wait for each to finish before spawning the next):

Instruct the subagent: "Follow the `ny-implement` skill. Your phase index is `<N>`, name is `<phase name>`, AC IDs are `<acIds>`. Read `.opencode/context.json` for the full context."

### 3. Lightweight verify

Read the branch name from `.opencode/context.json`. Detect this project's own lint, typecheck, and test commands from its configuration (e.g. `package.json` scripts, `Makefile`/`justfile`, or instructions in `AGENTS.md`) and run them in the **main pipeline thread** — not a subagent:

1. Lint
2. Typecheck
3. Full test suite

Zero-tolerance: if any check fails, fix the issue and re-run all three from the start. All must pass before proceeding.

**Completion**: all three checks pass with zero failures.

### 4. Code review (pre-PR, main thread)

Run the `/code-review` skill inline in the **main pipeline thread**, before any PR exists. Nothing is posted to GitHub in this step.

1. **Pin the fixed point**: review `git diff main...HEAD` (three-dot) plus `git log main..HEAD --oneline`. Verify the ref resolves (`git rev-parse main`) and the diff is non-empty before spawning anything — fail here, not inside the review agents.
2. **Spec source**: use the issue title/body already stored in `.opencode/context.json`.
3. **Standards sources**: whatever documents how *this* repo writes code — `AGENTS.md`/`CLAUDE.md`, `CONTRIBUTING.md`, files under `docs/`, ADRs, style guides.
4. **Fan out** two Task sub-agents **in parallel**:
   - **Standards**: give it the diff command, commit list, standards-source file list, and the full smell baseline pasted into the prompt (the sub-agent has no other access). Brief: report every place the diff violates a documented standard (cite file + rule) and any baseline smell (name it, quote the hunk); documented repo standards override baseline smells; label judgement calls as such; skip anything tooling enforces; under 400 words.
   - **Spec**: give it the diff command, commit list, and the spec text pasted in. Brief: report missing or partial requirements, behaviour that wasn't asked for (scope creep), and implementations that look wrong; quote the spec line per finding; under 400 words.
5. **Aggregate** both reports verbatim under `## Standards` and `## Spec` headings. End with a one-line summary: total findings per axis and the worst issue within each axis. Never rerank across axes.
6. Do **not** post anything to GitHub — there is no PR yet.

Store the aggregated report in `.opencode/context.json` as `codeReview` (merge, don't overwrite).

**Completion**: aggregated report presented to the user and saved in context.json.

### 5. Address findings (main thread)

Fix the findings from step 4 in the **main pipeline thread**:

1. Fix every finding:
   - Documented-standard breaches are mandatory fixes.
   - Baseline smells are judgement calls — fix unless a documented repo standard endorses the current shape ("the repo overrides"); skip anything tooling already enforces.
2. Re-run all three checks from step 3 (the project's lint, typecheck, and full test suite). Code changed after step 3, so the zero-tolerance gate re-arms here: if any check fails, fix and re-run all three from the start.
3. Single pass — do **not** re-run the code review after fixing.
4. Record the resolution per finding in `.opencode/context.json` under `codeReview` (e.g. findings with `resolved` flags).

**Completion**: findings addressed, all three checks green, resolutions recorded.

### 6. Open PR

Read issue number, title, branch name from `.opencode/context.json`. Do the following **in order**:

1. Commit and push all implementation changes:
   ```bash
   git add -A && git commit -m "#<issue>: <title>"
   git push origin "$(git rev-parse --abbrev-ref HEAD)"
   ```
2. Create the PR:
   ```bash
   gh pr create --title "Issue #<number>: <title>" --body "<acceptance criteria from issue body>" --base main
   ```
   Optionally prepend a one-line review summary (e.g. "Code review: N findings addressed.") to the PR body.
3. Capture the PR number from the output. Store it in `.opencode/context.json` as `prNumber`, plus the PR URL as `prUrl`.

**Completion**: PR is created and `prNumber` is in context.json.

### 7. Post-PR external review loop

Run **after** step 6 completes (`prNumber` present in `.opencode/context.json`). A second model reviews the open PR; the main thread fixes what it finds. Max **2 rounds**, then stop — this prevents infinite ping-pong between models.

Each round, in the **main pipeline thread**:

1. Spawn a Task subagent with type `pr-reviewer`. Pass it: the PR number and URL from context.json (the agent accepts either), the round number (1-based), and the issue title/body as the spec. The agent reads the repo itself; do not paste the diff into the prompt.
2. Wait for it to return its JSON verdict. If verdict is `ERROR`, follow error handling below.
3. If verdict is `APPROVE`: record the round under `externalReview` in `.opencode/context.json` (merge, don't overwrite) and stop — the pipeline is complete.
4. Otherwise address the returned findings in the **main thread**, mirroring step 5:
   - `mandatory: true` findings are required fixes.
   - `mandatory: false` findings are judgement calls — fix unless a documented repo standard endorses the current shape ("the repo overrides").
   - Do **not** re-run the internal code review from step 4.
5. Re-run all three checks from step 3 (the project's lint, typecheck, and full test suite). The zero-tolerance gate re-arms: if any check fails, fix and re-run all three from the start.
6. Commit and push: `git add -A && git commit -m "#<issue>: address external review round <N>" && git push origin "$(git rev-parse --abbrev-ref HEAD)"`.
7. Record per-finding resolutions and the round outcome under `externalReview` in `.opencode/context.json`.
8. Proceed to the next round (step 1) until APPROVE or 2 rounds completed. After the final round with verdict still `CHANGES_REQUESTED`, report the remaining findings to the user — they may warrant follow-up issues rather than more auto-fixing.

**Completion**: reviewer posted one comment per round, fixes pushed with green checks, `externalReview` recorded in context.json, loop ended via APPROVE or round limit.

## Error handling

If any subagent fails (steps 1–2, either review axis in step 4, or the `pr-reviewer` agent in step 7), retry it once. If it fails again, stop the pipeline and report the error. In step 7, a retry means re-spawning the same review round; findings already fixed in earlier rounds are not reverted.
