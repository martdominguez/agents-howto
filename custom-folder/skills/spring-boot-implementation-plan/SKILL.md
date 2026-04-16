---
name: Spring Boot Implementation Plan
description: Turn an approved impact analysis into an ordered implementation plan.
---

Use this skill after impact analysis is complete.

## Output structure

1. Preconditions
2. Files to modify
3. Ordered implementation steps
4. Migration strategy
5. Test plan
6. Rollback or fallback considerations

## Guidance

- Prefer the smallest safe sequence of changes.
- Call out places where temporary compatibility code may be needed.
- Mention whether DTO changes should be additive.
