Absolutely — here is a **ready-to-paste starter kit**.

I organized it so you can copy each file directly into your repo under `.github/...`.

---

# Suggested structure

```text
.github/
├─ copilot-instructions.md
├─ instructions/
│  ├─ backend.instructions.md
│  └─ tests.instructions.md
├─ agents/
│  ├─ architect.agent.md
│  ├─ backend-spring.agent.md
│  ├─ angular-ui.agent.md
│  ├─ mongo.agent.md
│  └─ reviewer.agent.md
├─ prompts/
│  ├─ analyze-ticket.prompt.md
│  ├─ generate-feature-docs.prompt.md
│  ├─ implement-step.prompt.md
│  └─ review-pr.prompt.md
└─ skills/
   ├─ spring-boot-standards/
   │  └─ SKILL.md
   └─ mongo-patterns/
      └─ SKILL.md
```

---

# 1) `.github/copilot-instructions.md`

```md
# Copilot Repository Instructions

## Project context

This repository contains an application based on:

- Spring Boot backend
- Angular frontend
- MongoDB persistence
- REST APIs documented with OpenAPI / Swagger
- Unit tests with JUnit 5 and Mockito

The purpose of these instructions is to make code generation and code review more consistent, practical, and aligned with the project architecture.

---

## General behavior

- Prefer practical implementation over theoretical explanations.
- Before making code changes, first analyze the requirement and identify impacted layers and files.
- Keep changes small, incremental, and reviewable.
- Reuse existing project patterns before introducing new abstractions.
- Do not perform large refactors unless explicitly requested.
- Preserve backward compatibility unless the requirement explicitly allows breaking changes.
- When implementing a feature, think first in terms of:
  1. API contract
  2. DTOs
  3. service logic
  4. persistence logic
  5. validation
  6. exception handling
  7. tests
  8. documentation

---

## Backend architecture expectations

For backend features, prefer the following layering:

- `controller`: HTTP entrypoints, request/response mapping, OpenAPI annotations
- `service`: business logic and orchestration
- `repository`: persistence access
- `repository custom implementation`: MongoTemplate, Criteria, Aggregation, complex queries
- `dto`: request and response payloads
- `entity` or `document`: persistence model
- `exception`: domain exceptions and error model

General backend rules:

- Use constructor injection.
- Use Lombok where appropriate.
- Avoid star imports.
- Prefer normal imports over fully qualified names inside code.
- Keep controllers thin.
- Do not expose persistence models directly in the API unless the existing project already does so intentionally.
- Prefer DTOs for API contracts.
- Reuse an existing `@ControllerAdvice` if present.
- Add OpenAPI annotations on public endpoints.
- Validate request DTOs with Jakarta Validation when appropriate.
- Keep business rules in the service layer, not in controllers.
- Keep persistence-specific logic out of services when it belongs in custom repositories.

---

## MongoDB expectations

When working with MongoDB:

- Prefer standard Spring Data repositories for simple CRUD.
- Prefer custom repositories with `MongoTemplate` for non-trivial queries.
- Prefer programmatic Criteria / Query / Aggregation code for complex cases.
- Write aggregation pipelines in a readable way.
- Be explicit about null handling, missing fields, empty arrays, and type conversions.
- If a query is complex, keep the code maintainable rather than overly compact.
- When appropriate, mention indexing assumptions or performance considerations in comments or documentation.

---

## Angular expectations

When working on Angular code:

- Follow existing project structure and naming conventions.
- Keep services responsible for HTTP communication.
- Keep components focused on presentation and UI orchestration.
- Avoid putting large logic blocks directly in templates.
- Keep TypeScript types explicit for backend contracts.
- Reuse existing shared components, interceptors, utilities, and styles if available.
- Prefer incremental UI changes over broad rewrites.

---

## Testing expectations

When creating or updating tests:

- Use JUnit 5 and Mockito for backend tests.
- Prefer parameterized tests when they improve readability and remove duplication.
- Use a clear `given / when / then` structure.
- Test observable behavior rather than implementation details.
- Mock collaborators, not simple data objects.
- Do not over-mock.
- Add tests for edge cases when business rules require them.
- When changing controllers, consider request validation and error scenarios.
- When changing services, consider happy path and failure path behavior.
- When changing Mongo custom logic, test the service behavior that depends on that logic, and add repository-focused tests if the project already has that pattern.

---

## API design expectations

For REST endpoints:

- Use clear and stable naming.
- Keep request and response DTOs explicit.
- Use appropriate HTTP status codes.
- Document endpoints with OpenAPI annotations.
- Return consistent error responses through the project's existing error-handling mechanism.
- If the requirement is ambiguous, prefer practical consistency with the existing API style in the repository.

---

## Workflow expectations

When asked to implement a requirement:

1. Analyze the requirement first.
2. Identify impacted modules and files.
3. Propose a small implementation plan.
4. Implement one logical step at a time.
5. Review for gaps.
6. Add or update tests.
7. Update documentation when needed.

When asked to review code:

- Be strict but practical.
- Look for correctness, clarity, maintainability, consistency, test quality, and unnecessary complexity.
- Distinguish between:
  - blocking issues
  - non-blocking improvements
  - optional nice-to-have suggestions

---

## Output style expectations

- Be concise but precise.
- When planning, use ordered steps.
- When reviewing, provide actionable feedback.
- When generating code, align with the repository's existing package structure and naming.
- Avoid unnecessary commentary inside code.
- Prefer readable code over clever code.
```

---

# 2) `.github/instructions/backend.instructions.md`

```md
# Backend Instructions

Apply these instructions when working on Spring Boot backend code.

## General backend approach

- Follow the existing package structure and conventions of the project.
- Keep controllers thin.
- Keep business logic in services.
- Keep persistence logic in repositories or custom repository implementations.
- Prefer DTOs for API contracts.
- Reuse existing patterns before creating new abstractions.

## Dependency injection and object creation

- Use constructor injection.
- Use Lombok where appropriate.
- Prefer builder pattern when it improves object construction readability.
- Avoid field injection.

## Imports and code style

- Avoid star imports.
- Prefer regular imports over fully qualified names in code.
- Keep methods reasonably small and focused.
- Prefer descriptive names over abbreviations.

## REST controller rules

- Add OpenAPI annotations for public endpoints.
- Use request DTOs and response DTOs.
- Validate incoming payloads with Jakarta Validation when appropriate.
- Keep HTTP concerns in the controller layer only.

## Service layer rules

- Put business decisions and orchestration in services.
- Do not push business logic into controllers.
- Do not embed complex persistence code directly in services when it belongs in a custom repository.

## Repository rules

- Use standard Spring Data repositories for basic CRUD.
- For custom search, filtering, updates, or aggregation logic, prefer custom repository implementations.
- When using MongoDB with non-trivial logic, prefer MongoTemplate and programmatic Criteria/Aggregation code.

## Exception handling

- Reuse the existing `@ControllerAdvice` if available.
- Reuse existing exception patterns before adding new ones.
- Add new domain exceptions only when they clearly improve expressiveness and consistency.

## Documentation and compatibility

- Keep OpenAPI documentation aligned with the implemented contract.
- Avoid breaking existing API contracts unless explicitly requested.
- If behavior changes, update related DTOs, annotations, and error handling consistently.

## Practical expectations

When implementing a backend requirement, prefer this sequence:

1. request/response DTOs
2. controller contract
3. service logic
4. repository/custom repository logic
5. exceptions and error mapping
6. tests
7. documentation updates
```

---

# 3) `.github/instructions/tests.instructions.md`

```md
# Test Instructions

Apply these instructions when creating or editing tests.

## General testing principles

- Use JUnit 5 and Mockito.
- Prefer readability over excessive cleverness.
- Test behavior, not internal implementation details.
- Keep each test focused on one scenario.
- Cover both expected behavior and relevant failure scenarios.

## Structure

Prefer a clear given / when / then structure in each test.

When useful, use comments or display names that make the intent explicit.

Example expectation:

- given: initial state, mocks, inputs
- when: invoke the method under test
- then: assert result and verify relevant interactions

## Parameterized tests

- Prefer parameterized tests when they reduce duplication and keep the test readable.
- Do not force parameterized tests if they make the scenario harder to understand.

## Mockito usage

- Mock collaborators, not simple DTOs or value objects.
- Verify only interactions that matter to the behavior being tested.
- Avoid over-verifying every call.
- Prefer stubbing only what is necessary.

## Service tests

For service tests, focus on:

- business rules
- branching behavior
- exception scenarios
- interaction with repositories or external collaborators

## Controller tests

For controller tests, focus on:

- request mapping
- validation behavior
- response status
- response payload shape
- error scenarios handled through the application's error-handling strategy

## Repository-related tests

If the project uses tests for custom repository behavior:

- focus on query intent and observable results
- keep tests maintainable
- avoid unnecessary coupling to internal implementation details

## Naming

- Use descriptive test names.
- Prefer names that explain behavior and expected outcome.

## Final expectation

Every test added or modified should help answer one of these questions:

- does the feature work?
- does the rule behave correctly?
- does the application fail correctly in invalid scenarios?
- is the contract protected from regressions?
```

---

# 4) Agents

## `.github/agents/architect.agent.md`

```md
---
description: Analyze requirements and produce implementation-ready feature documentation.
tools: inherit
---

You are a senior software architect and technical writer.

Your role is to transform requirements into structured, implementation-ready documentation.

## Your responsibilities

- Analyze requirements from Jira tickets, markdown notes, or user prompts.
- Identify functional and non-functional requirements.
- Identify assumptions, edge cases, risks, and missing information.
- Propose a practical design aligned with the repository architecture.
- Produce documentation before implementation when asked.

## Your default output style

Prefer structured markdown with sections such as:

1. functional summary
2. impacted areas
3. assumptions
4. edge cases
5. technical approach
6. implementation steps
7. validation checklist

## Important rules

- Do not implement code unless explicitly asked.
- Prefer practical decisions over abstract discussion.
- Align your reasoning with Spring Boot + Angular + MongoDB architecture.
- Be precise and implementation-oriented.
- When proposing changes, keep them small and incremental.
```

---

## `.github/agents/backend-spring.agent.md`

```md
---
description: Implement Spring Boot backend features in small, testable increments.
tools: inherit
---

You are a senior Spring Boot backend engineer.

You work on a backend based on:
- Spring Boot
- MongoDB
- MongoTemplate
- REST API with OpenAPI
- JUnit 5 and Mockito

## Your responsibilities

- Implement backend changes in small, safe steps.
- Follow repository instructions and backend instructions.
- Keep architecture layered and consistent.
- Reuse existing patterns before introducing new ones.
- Keep code readable, explicit, and maintainable.

## Default workflow

Before coding:
1. analyze the requirement
2. identify impacted files
3. state assumptions if needed
4. implement only the requested step

When coding:
- keep controllers thin
- put business rules in services
- put complex persistence logic in custom repositories
- use DTOs for API contracts
- reuse existing exception handling patterns
- update OpenAPI annotations when endpoint contracts change

## Important rules

- Do not make unrelated refactors.
- Do not implement everything at once unless explicitly asked.
- Prefer incremental changes.
- Add or update tests when relevant.
- If a requirement is ambiguous, choose the most consistent option with the current repository structure.
```

---

## `.github/agents/angular-ui.agent.md`

```md
---
description: Implement Angular UI changes aligned with backend contracts and existing frontend patterns.
tools: inherit
---

You are a senior Angular engineer.

The frontend works with a Spring Boot backend and typed REST contracts.

## Your responsibilities

- Implement Angular changes in a way that fits the current frontend structure.
- Keep components focused and maintainable.
- Keep services responsible for backend communication.
- Reuse existing UI patterns, routing conventions, and shared utilities.

## Important rules

- Do not rewrite broad areas of the UI unless explicitly requested.
- Prefer incremental changes.
- Keep TypeScript models aligned with backend DTOs.
- Avoid pushing too much logic into templates.
- Keep user interactions and state transitions understandable.

## When implementing

Prefer this order:
1. DTO / interface alignment
2. service integration
3. component logic
4. template update
5. validation and user feedback
6. tests if applicable
```

---

## `.github/agents/mongo.agent.md`

```md
---
description: Design and implement MongoDB repository logic using Spring Data and MongoTemplate.
tools: inherit
---

You are a senior MongoDB and Spring Data engineer.

Your focus is:
- Spring Data repositories
- custom repositories
- MongoTemplate
- Query, Criteria, Update, Aggregation
- readable and maintainable persistence logic

## Your responsibilities

- Design and implement Mongo persistence logic.
- Use standard repositories for simple CRUD.
- Use custom repository implementations for non-trivial queries and aggregations.
- Keep pipelines readable and explicit.
- Think about correctness first, then maintainability, then performance.

## Important rules

- Prefer programmatic aggregation/query code for complex cases.
- Be explicit about null values, missing fields, empty arrays, and type conversions.
- Avoid over-compacting query logic.
- If relevant, mention indexing or performance assumptions.
- Keep Mongo-specific logic out of controllers.

## Output expectations

When asked to propose or implement Mongo logic:
- explain the query intent
- identify the best layer for the logic
- implement or outline the query clearly
- mention edge cases that affect results
```

---

## `.github/agents/reviewer.agent.md`

```md
---
description: Review code as a strict senior teammate before PR or merge.
tools: inherit
---

You are a strict but practical senior reviewer.

You review code for:
- correctness
- maintainability
- clarity
- architecture consistency
- API safety
- Mongo query quality
- test quality
- error handling quality

## Review style

Always distinguish between:

1. Blocking issues
2. Non-blocking improvements
3. Nice-to-have suggestions

## What to look for

- Is the implementation correct?
- Is the design aligned with project conventions?
- Is complexity justified?
- Are there contract risks?
- Are validations correct?
- Are exceptions handled consistently?
- Are tests meaningful and sufficient?
- Is there unnecessary duplication?
- Is Mongo logic readable and correctly placed?

## Important rules

- Be direct and actionable.
- Do not praise unnecessarily.
- Prefer specific recommendations over vague comments.
- Keep the feedback practical and usable in a PR workflow.
```

---

# 5) Prompt files

## `.github/prompts/analyze-ticket.prompt.md`

```md
Analyze this requirement as a senior architect.

Inputs may include:
- Jira ticket text
- Confluence notes
- existing code context
- API contract notes

Produce:

1. Functional summary
2. Non-functional considerations
3. Impacted backend areas
4. Impacted frontend areas
5. Edge cases
6. Missing information or assumptions
7. Recommended technical approach
8. Suggested implementation slices

Rules:
- Do not implement code yet
- Prefer practical decisions
- Align with Spring Boot + Angular + MongoDB
- Keep the result structured and implementation-oriented
```

---

## `.github/prompts/generate-feature-docs.prompt.md`

```md
You are a senior software architect and technical writer.

Generate a complete documentation set for this feature under:

docs/requirements/<ID-feature-name>/

Create these files:

1. requerimiento.md
2. analisis-funcional.md
3. analisis-tecnico.md
4. plan-implementacion.md
5. prompt-implementacion.md
6. validacion.md

Rules:
- Do NOT implement code
- Use clear and structured markdown
- Be concise but precise
- Prefer practical decisions over theoretical explanations
- Assume a Spring Boot backend, Angular frontend, MongoDB persistence, and REST API with OpenAPI
- The implementation plan must be incremental and testable
- The validation file must include both functional and technical checks
```

---

## `.github/prompts/implement-step.prompt.md`

```md
Implement only the requested step of the feature.

Before coding:
1. summarize the goal of the step
2. list impacted files
3. state assumptions if needed

Then implement the change.

Constraints:
- keep the change minimal but complete
- align with the repository instructions
- follow layered architecture
- reuse existing patterns first
- if backend contract changes, update OpenAPI annotations and related DTOs
- if relevant, add or update tests

After coding, provide:
1. summary of what changed
2. potential follow-up step
3. any remaining risks or assumptions
```

---

## `.github/prompts/review-pr.prompt.md`

```md
Review the current implementation like a strict senior teammate.

Focus on:
- correctness
- readability
- maintainability
- architecture consistency
- API contract safety
- Mongo query quality
- validation behavior
- exception handling
- test quality

Output format:

## Blocking issues
- ...

## Non-blocking improvements
- ...

## Nice-to-have suggestions
- ...

Rules:
- be direct
- be actionable
- avoid vague feedback
- prefer practical recommendations
```

---

# 6) Skills

## `.github/skills/spring-boot-standards/SKILL.md`

```md
---
name: Spring Boot Standards
description: Use this skill when implementing or reviewing Spring Boot backend code, REST endpoints, DTOs, services, exception handling, and tests.
---

# Spring Boot Standards Skill

Use this skill when working on backend code for this repository.

## Goals

- Keep backend code consistent
- Keep layers well separated
- Keep APIs explicit and documented
- Keep tests meaningful
- Keep changes easy to review

---

## Package and layering guidance

Typical layers:

- controller
- service
- repository
- repository custom implementation
- dto
- entity or document
- exception

### Layer responsibilities

#### Controller
Responsible for:
- endpoint mapping
- request validation entrypoint
- request/response DTO handling
- OpenAPI annotations
- translating service output into HTTP response if needed

Should avoid:
- business logic
- query logic
- persistence decisions

#### Service
Responsible for:
- business rules
- orchestration
- validation beyond simple DTO annotations if business-specific
- calling repositories and collaborators
- deciding domain outcomes

Should avoid:
- direct HTTP concerns
- complex Mongo query construction if it belongs in a repository

#### Repository
Responsible for:
- persistence access

Use:
- standard repository for simple CRUD
- custom repository implementation for complex query logic

---

## DTO guidance

Use DTOs for API contracts.

### Request DTOs
Prefer:
- validation annotations where relevant
- focused fields
- clear names

### Response DTOs
Prefer:
- explicit fields
- stable contract shape
- not leaking persistence concerns

---

## OpenAPI guidance

For public endpoints:
- add clear summary
- add clear description when useful
- document request body
- document response codes
- document error responses if the project already follows that pattern

---

## Exception handling guidance

Prefer:
- domain-specific exceptions when useful
- reusing existing exception hierarchy
- reusing existing `@ControllerAdvice`

Do not:
- create ad hoc error handling inside each controller unless the project explicitly does that

---

## Code style guidance

- constructor injection
- Lombok where appropriate
- avoid star imports
- use clear naming
- prefer readable code over compact cleverness

---

## Testing guidance

### Service tests
Usually test:
- happy path
- branching path
- exception path

### Controller tests
Usually test:
- status codes
- validation
- response payload
- error handling integration

### Mockito
Prefer:
- focused stubbing
- focused verification
- no over-mocking

### Parameterized tests
Use when:
- the same rule is exercised with multiple input/output combinations
- readability improves

---

## Practical implementation checklist

Before implementation:
- understand requirement
- identify impacted layers
- decide DTOs
- decide service responsibilities
- decide repository/custom repository responsibilities

Before finishing:
- endpoint documented
- validation aligned
- exceptions handled
- tests updated
- no unrelated refactor introduced

---

## Example implementation order

1. request/response DTOs
2. controller contract
3. service logic
4. repository/custom repository logic
5. exceptions
6. tests
7. docs update
```

---

## `.github/skills/mongo-patterns/SKILL.md`

```md
---
name: Mongo Patterns
description: Use this skill when implementing or reviewing MongoDB persistence logic with Spring Data, MongoTemplate, Criteria, Query, Update, and Aggregation.
---

# Mongo Patterns Skill

Use this skill when the feature requires MongoDB access beyond trivial CRUD.

## Goals

- Keep Mongo logic correct
- Keep query code readable
- Put logic in the right layer
- Make complex queries maintainable

---

## When to use standard repository vs custom repository

### Use standard Spring Data repository when:
- CRUD is simple
- derived query methods are enough
- query complexity is low

### Use custom repository + MongoTemplate when:
- query conditions are dynamic
- aggregation is needed
- update logic is custom
- projection logic is custom
- readability would suffer with annotation-based queries

---

## Recommended placement

- simple CRUD: repository interface
- complex query or aggregation: custom repository implementation
- orchestration and business meaning: service
- request handling: controller

Do not put Mongo query construction in controllers.

---

## Query design guidance

When building queries:
- be explicit about optional criteria
- be explicit about null handling
- be explicit about missing fields
- be explicit about empty arrays
- be explicit about sorting and pagination when relevant

Prefer readable, step-by-step query construction over overly condensed code.

---

## Aggregation guidance

When writing an aggregation pipeline:
- keep each stage understandable
- prefer programmatic construction for complex pipelines
- add comments if a stage is not self-evident
- think about how null or missing values affect the result
- think about whether projection, unwind, group, sort, and match order are correct

Good aggregation code should be easy to debug and review.

---

## Update guidance

For updates:
- keep update intent explicit
- avoid accidental partial updates or field loss
- be careful with arrays, nested fields, and absent fields

---

## Performance and maintainability guidance

When relevant, consider:
- whether the query can use an index
- whether the aggregation will scan too much data
- whether filtering can be moved earlier in the pipeline
- whether the projection can reduce payload size

Do not prematurely optimize, but do not ignore obvious performance risks.

---

## Edge cases checklist

Always think about:
- null field
- missing field
- empty array
- duplicated values
- unexpected type
- date/time comparison behavior
- sort behavior with nulls
- partial data

---

## Practical implementation checklist

Before implementation:
- identify if CRUD is simple or not
- decide whether custom repository is needed
- identify dynamic filters
- identify result shape

Before finishing:
- query intent is clear
- edge cases considered
- correct layer used
- service behavior tested
- no Mongo logic leaked into controller
```

---

# 7) How to use this in practice

A very good real flow would be:

### For a new ticket

Use `architect.agent.md` with `analyze-ticket.prompt.md`

### Then

Use `generate-feature-docs.prompt.md`

### Then

Use `backend-spring.agent.md` or `angular-ui.agent.md` with `implement-step.prompt.md`

### Then

Use `mongo.agent.md` for repository/query work

### Finally

Use `reviewer.agent.md` with `review-pr.prompt.md`
