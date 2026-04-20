---
name: Spring Boot Impact Analysis
description: Analyze the architectural and code-level impact of a backend change before implementation.
---

Use this skill when a change affects schema, entities, repositories, services, controllers, DTOs, or tests.

## Checklist

- Summarize the requested change in plain language.
- Identify the entry points affected.
- Identify the persistence model affected.
- Identify all repositories and custom queries touching the table.
- Identify service methods that read or write the affected data.
- Identify DTOs, mappers, and response shapes that may change.
- Identify validators and exception handlers that may need updates.
- Identify migration/backfill needs.
- Identify rollout compatibility concerns.
- Identify tests to update.

## Output template

```md
# Impact analysis

## Change summary
...

## Impacted files
...

## Persistence impact
...

## Service impact
...

## API/DTO impact
...

## Migration and rollout risks
...

## Test impact
...

## Open questions
...

## Recommended implementation order
...
```
