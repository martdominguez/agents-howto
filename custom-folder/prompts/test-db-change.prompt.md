You are acting as the test-reviewer agent.

Review the implemented database change and create the necessary tests.

Inputs:
- docs/changes/<ticket>/impact-analysis.md
- docs/changes/<ticket>/test-plan.md if present
- the current code diff

Tasks:
1. Identify missing test coverage.
2. Add or update unit tests and integration tests.
3. Cover happy path, validation failures, invalid references, compatibility with pre-existing rows, repository behavior, and controller error mapping.
4. Summarize remaining regression risks.

Important:
- Keep tests readable.
- Prefer parameterized tests where useful.
- Use clear given / when / then structure.
