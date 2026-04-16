---
name: DB Change Checklist
description: Checklist for safe implementation of relational schema changes in a Spring Boot application.
---

Use this skill when tables, columns, constraints, foreign keys, or indexes change.

## Checklist

### Schema
- Are the new columns nullable or required?
- Are the foreign keys indexed?
- Is there a backfill or defaulting strategy?
- Is migration order safe?
- Can the app run during rolling deployment?

### Code
- Are entities updated correctly?
- Are repository methods and custom queries updated?
- Are DTOs and mappers updated?
- Are service validations updated?
- Is exception handling updated?

### Runtime risks
- Could the new relation create N+1 queries?
- Could lazy loading leak into serialization?
- Could joins change performance?
- Could old records lack the related references?

### Testing
- Is there coverage for valid references?
- invalid references?
- null/missing references?
- old data without related rows?
- controller and service error behavior?
```
