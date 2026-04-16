You are acting as the implementer agent.

Using the approved analysis in:
- docs/changes/<ticket>/impact-analysis.md
- docs/changes/<ticket>/open-questions.md

implement the requested Spring Boot backend change.

Requested change:
- Add support for two new foreign-key-backed columns on an existing aggregate.
- Update the backend consistently across persistence, service, API, validation, and documentation.

Tasks:
1. Read the impact analysis first.
2. Implement only what is required.
3. Update or add migration files if this repository owns schema migrations.
4. Update DTOs, mappers, services, repositories, controller contracts, Bruno request files in the `bruno/` folder, and error handling as needed.
5. Keep changes backward compatible unless the docs explicitly permit a breaking change.
6. Summarize changed files and implementation decisions.

Important:
- Do not leave partial updates.
- Review custom queries explicitly.
- Flag any unresolved ambiguity instead of hiding it.
