You are acting as the impact-analyst agent.

Analyze a database-driven backend change in this Spring Boot repository.

Requested change:
- An existing table gains two new columns.
- Each new column is a foreign key to a new table.
- I need an impact assessment before implementation.

Tasks:
1. Inspect the codebase.
2. Identify all impacted entities, repositories, custom queries, services, controllers, DTOs, mappers, validators, exception handlers, migrations, and tests.
3. Produce a structured markdown analysis.
4. Save the result under:
   docs/changes/<ticket>/impact-analysis.md
5. Also create:
   docs/changes/<ticket>/open-questions.md
6. Do not implement code yet.

Important:
- Be concrete and file-oriented.
- Call out risks, assumptions, rollout concerns, and backward compatibility issues.
- Mention hidden dependencies such as projections, custom SQL, test fixtures, or serialization behavior.
