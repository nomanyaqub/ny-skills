---
name: ny-document
description: "Update project documentation for user-facing changes: docs/, README.md, .env.example, and docs/testing.md, verifying each change against the actual code. Use when documentation must reflect implemented changes."
---

# ny-document

Update project documentation to match the current implementation:

1. Update `docs/` or `README.md` if there are user-facing changes
2. Sync `.env.example` if environment variables changed
3. Update `docs/testing.md`:
   - Document curl commands for new API endpoints
   - Add Hurl test files to the file list
   - Update test counts

Verify each change against the actual code — do not update from memory.

**Completion**: all three tasks verified against the current implementation.
