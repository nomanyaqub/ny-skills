---
name: ny-coverage
---

# ny-coverage

**Hunt** for test gaps discovered during audit. Add tests for every gap found:

- Unit tests for missing edge cases
- Hurl tests in `tests/api/` for end-to-end coverage when applicable

For each gap, either add a test or document a decision not to cover it (with rationale).

**Completion**: every gap from audit has a corresponding test or a documented decision to skip.
