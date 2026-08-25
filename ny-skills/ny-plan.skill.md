---
name: ny-plan
---

# ny-plan

Read `.opencode/context.json` to understand the issue. Read `.opencode/fetched-issue.json` for the full issue body.

1. Study the relevant code in the project
2. Present a clear plan covering:
   - What files need to change
   - The approach and design
   - Any decisions to make
   - When the plan touches multiple areas, split into independent phases
3. Parse acceptance criteria from the issue body (look for `- [ ]` checklist items). For each, assign which phase it belongs to. If a criterion spans multiple phases or is purely meta (linting, docs), assign it to all relevant phases.
4. **Stop and wait** for the user to say "go ahead" (or similar) before proceeding
5. If approved, write into `.opencode/context.json`:
   ```json
   {
     "issueNumber": ...,
     "phases": [
       { "name": "Phase 1: ...", "description": "...", "done": false, "acIds": [0, 1] }
     ],
     "acceptanceCriteria": [
       { "id": 0, "text": "...", "status": "pending" }
     ]
   }
   ```
   - `phases` — each has a `name`, `description`, `done` flag, and `acIds` (list of acceptance criterion IDs covered by this phase)
   - `acceptanceCriteria` — each has an `id`, the exact `text` from the issue, and `status` (one of `"pending"`, `"verified"`)
6. If rejected, abort

**Completion**: context.json is populated with phases and acceptance criteria, or the pipeline is aborted.
