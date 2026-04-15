GitHub Copilot now supports 
- repository-wide instructions via `.github/copilot-instructions.md`; 
- path-specific instructions via `.github/instructions/**/*.instructions.md`; 
- custom CLI agents via `.agent.md` files; 
- agent skills as folders with instructions/scripts/resources; 
- and prompt files in VS Code under `.github/prompts`. 
- Copilot CLI is GA, and custom agents can be stored either per-repo or in your user profile. ([GitHub Docs][1])

# 1) Starter kit repo structure

This is the version I would actually use in a team repo.

```text
repo-root/
├─ .github/
│  ├─ copilot-instructions.md
│  ├─ instructions/
│  │  ├─ backend.instructions.md
│  │  ├─ frontend.instructions.md
│  │  ├─ tests.instructions.md
│  │  ├─ mongodb.instructions.md
│  │  ├─ api-review.instructions.md
│  │  └─ docs.instructions.md
│  ├─ agents/
│  │  ├─ architect.agent.md
│  │  ├─ backend-spring.agent.md
│  │  ├─ angular-ui.agent.md
│  │  ├─ mongo.agent.md
│  │  ├─ tester.agent.md
│  │  └─ reviewer.agent.md
│  ├─ prompts/
│  │  ├─ analyze-ticket.prompt.md
│  │  ├─ generate-feature-docs.prompt.md
│  │  ├─ create-implementation-plan.prompt.md
│  │  ├─ implement-backend-step.prompt.md
│  │  ├─ implement-frontend-step.prompt.md
│  │  ├─ validate-feature.prompt.md
│  │  ├─ review-pr.prompt.md
│  │  └─ refactor-safely.prompt.md
│  └─ skills/
│     ├─ spring-boot-standards/
│     │  ├─ SKILL.md
│     │  ├─ examples/
│     │  │  ├─ controller-example.md
│     │  │  ├─ service-example.md
│     │  │  └─ controller-advice-example.md
│     │  └─ checklists/
│     │     └─ implementation-checklist.md
│     ├─ mongo-patterns/
│     │  ├─ SKILL.md
│     │  ├─ examples/
│     │  │  ├─ aggregation-example.md
│     │  │  ├─ criteria-example.md
│     │  │  └─ custom-repository-example.md
│     │  └─ checklists/
│     │     └─ mongo-review-checklist.md
│     ├─ api-design/
│     │  ├─ SKILL.md
│     │  ├─ examples/
│     │  │  ├─ openapi-controller-example.md
│     │  │  └─ dto-contract-example.md
│     │  └─ checklists/
│     │     └─ api-contract-checklist.md
│     └─ testing-junit-mockito/
│        ├─ SKILL.md
│        ├─ examples/
│        │  ├─ unit-test-example.md
│        │  ├─ parameterized-test-example.md
│        │  └─ given-when-then-example.md
│        └─ checklists/
│           └─ test-review-checklist.md
│
├─ docs/
│  ├─ architecture/
│  │  ├─ overview.md
│  │  ├─ backend-architecture.md
│  │  ├─ frontend-architecture.md
│  │  ├─ integration-patterns.md
│  │  └─ coding-standards.md
│  ├─ requirements/
│  │  ├─ 001-sample-feature/
│  │  │  ├─ requerimiento.md
│  │  │  ├─ analisis-funcional.md
│  │  │  ├─ analisis-tecnico.md
│  │  │  ├─ plan-implementacion.md
│  │  │  ├─ prompt-implementacion.md
│  │  │  └─ validacion.md
│  │  └─ ...
│  ├─ api/
│  │  ├─ endpoint-catalog.md
│  │  └─ error-handling.md
│  └─ runbooks/
│     ├─ local-dev.md
│     ├─ testing.md
│     └─ pr-checklist.md
│
├─ backend/
│  └─ ...
├─ frontend/
│  └─ ...
└─ README.md
```

## Why this structure works

* `copilot-instructions.md` is your always-on baseline for the whole repo. ([GitHub Docs][1])
* `.github/instructions/*.instructions.md` lets you steer Copilot differently for backend, frontend, tests, docs, and Mongo-heavy code paths. ([GitHub Docs][2])
* `.github/agents/*.agent.md` gives you reusable specialist roles in Copilot CLI and other supported surfaces. ([GitHub Docs][3])
* `.github/prompts/*.prompt.md` is ideal for repeatable workflows in VS Code, especially analysis, planning, validation, and PR review. ([Visual Studio Code][4])
* `.github/skills/` is where you encode deep project knowledge that should be loaded only when relevant, rather than forcing everything into one giant global instruction file. ([GitHub Docs][5])

---

# 2) What each file should contain

## `.github/copilot-instructions.md`

This is the **stable repo contract**.

Use it for:

* tech stack summary
* architecture shape
* naming rules
* build/test commands
* high-level coding standards
* “how to behave in this repo”

A good version for you would look like this:

```md
# Copilot repository instructions

## Project overview
This repository contains:
- Spring Boot backend
- Angular frontend
- MongoDB for document persistence
- OpenAPI/Swagger-documented REST APIs

## General behavior
- Prefer practical implementation over theoretical discussion.
- Before coding, analyze the requirement and propose the impacted files.
- Keep changes small, reviewable, and aligned with the existing architecture.
- Do not introduce unnecessary frameworks or abstractions.
- Preserve backward compatibility unless the requirement explicitly allows breaking changes.

## Backend standards
- Use constructor injection.
- Use Lombok where appropriate.
- Avoid star imports.
- Prefer explicit imports over fully qualified names in code.
- Use clear package boundaries: controller, service, repository, dto, entity, mapper, exception.
- Add OpenAPI annotations for public REST endpoints.
- Reuse existing ControllerAdvice for error handling when possible.
- Custom Mongo queries and aggregations should be implemented programmatically rather than annotation-heavy if complexity increases.

## Frontend standards
- Prefer standalone, understandable Angular components and services.
- Keep API contracts typed.
- Keep UI changes aligned with existing routing, menu, and page composition patterns.

## Testing standards
- Prefer parameterized tests where useful.
- Use given-when-then structure.
- Keep tests focused on one behavior.
- Mock only true collaborators.
- Do not over-mock simple value objects or DTOs.

## Validation before finishing
Always verify:
1. contract compatibility
2. error scenarios
3. build correctness
4. test coverage impact
5. documentation updates
```

---

## Path-specific instruction files

These should be shorter and sharper than the global file.

### `backend.instructions.md`

Put backend-only rules there:

```md
Apply these instructions when working on backend code.

- Use Spring Boot conventions already present in the project.
- Expose REST endpoints with clear DTOs, not entities.
- Keep controller thin, service responsible for business logic, repository responsible for persistence concerns.
- Prefer MongoTemplate/custom repository for non-trivial Mongo queries and aggregations.
- Add validation annotations to request DTOs where appropriate.
- Reuse existing exception model and ControllerAdvice.
- Add or update OpenAPI annotations when endpoint behavior changes.
```

### `frontend.instructions.md`

```md
Apply these instructions when working on frontend code.

- Respect current Angular project structure and naming.
- Keep service layer responsible for backend communication.
- Do not hardcode backend URLs when existing environment configuration exists.
- Keep UI logic out of templates when possible.
- Prefer incremental UI changes over large rewrites.
```

### `tests.instructions.md`

```md
Apply these instructions when creating or editing tests.

- Use JUnit 5 and Mockito project conventions.
- Prefer parameterized tests when they improve readability.
- Use given-when-then sections clearly.
- Test observable behavior, not implementation details.
- Avoid brittle assertions on logs or internal call counts unless behavior requires it.
```

### `mongodb.instructions.md`

```md
Apply these instructions when working on MongoDB code.

- Prefer programmatic aggregations for non-trivial pipelines.
- Keep repository interfaces simple; move custom logic to custom repository implementation.
- Make pipeline stages readable and maintainable.
- Document assumptions about indexes or large-collection behavior if relevant.
- Be explicit about null, missing fields, empty arrays, and type conversions.
```

---

# 3) Custom agents I would define

GitHub supports custom agents in Markdown agent profile files, and those can specify instructions, tools, and specialized behavior. ([GitHub Docs][3])

## `architect.agent.md`

Use for requirement analysis and docs-first design.

```md
---
description: Analyze tickets and produce implementation-ready feature documentation.
tools: inherit
---

You are a senior software architect and technical writer.

Your job:
- Analyze requirement inputs from Jira, Confluence, and repository context
- Produce structured feature documentation
- Identify missing assumptions, edge cases, dependencies, risks
- Do not implement code unless explicitly asked

Preferred outputs:
- requerimiento.md
- analisis-funcional.md
- analisis-tecnico.md
- plan-implementacion.md
- validacion.md
```

## `backend-spring.agent.md`

Use for implementation steps.

```md
---
description: Implement Spring Boot backend features in small, testable increments.
tools: inherit
---

You are a senior Spring Boot engineer.

Rules:
- Implement one step at a time
- Follow repository instructions and backend path instructions
- Keep architecture layered
- Add DTOs, service, repository, controller, exceptions, and docs as needed
- Reuse existing patterns before introducing new ones
- Explain impacted files before patching
```

## `mongo.agent.md`

Use for queries and aggregations.

```md
---
description: Design and implement MongoTemplate queries, criteria, and aggregations.
tools: inherit
---

You are a MongoDB and Spring Data expert.

Focus on:
- custom repositories
- MongoTemplate
- aggregation pipelines
- performance-aware queries
- maintainable Java code over annotation-heavy shortcuts
```

## `tester.agent.md`

Use after each implementation step.

```md
---
description: Create and improve tests for backend and frontend changes.
tools: inherit
---

You are a strict but pragmatic test engineer.

Rules:
- Prefer behavior-focused tests
- Minimize brittle mocks
- Suggest missing edge cases
- Use existing project conventions first
```

## `reviewer.agent.md`

Use before PR or on review.

```md
---
description: Review code as a senior teammate before PR.
tools: inherit
---

You are a senior reviewer.

Review for:
- correctness
- maintainability
- unnecessary complexity
- contract safety
- test quality
- error handling
- alignment with repository standards

Be direct and actionable.
```

---

# 4) Skills I would create first

Skills are best for reusable domain-specific know-how, with examples and checklists, instead of dumping everything into one always-on instruction file. ([GitHub Docs][5])

## Skill 1: `spring-boot-standards`

Contents:

* controller conventions
* DTO naming
* service responsibilities
* exception strategy
* OpenAPI expectations
* package structure
* examples of “good” controller/service/advice

## Skill 2: `mongo-patterns`

Contents:

* custom repository patterns
* Criteria queries
* aggregation patterns
* update patterns
* handling empty arrays/nulls
* example pipelines

## Skill 3: `api-design`

Contents:

* endpoint naming
* request/response design
* error payload conventions
* pagination/filtering style
* status code rules
* compatibility checks

## Skill 4: `testing-junit-mockito`

Contents:

* unit test structure
* parameterized tests
* given/when/then style
* controller vs service test guidance
* mocking guidelines

---

# 5) Prompt files you should keep ready

Prompt files are especially useful in VS Code for repeatable tasks. ([Visual Studio Code][4])

## `analyze-ticket.prompt.md`

```md
Analyze this ticket as a senior architect.

Inputs:
- Jira ticket text
- Related Confluence notes
- Existing codebase patterns

Produce:
1. Functional summary
2. Impacted areas
3. Missing information
4. Edge cases
5. Risks
6. Recommended implementation strategy

Do not implement code yet.
```

## `generate-feature-docs.prompt.md`

```md
Generate a complete feature documentation set under:

docs/requirements/<ID-feature-name>/

Create:
- requerimiento.md
- analisis-funcional.md
- analisis-tecnico.md
- plan-implementacion.md
- prompt-implementacion.md
- validacion.md

Rules:
- Do not write code
- Be implementation-oriented
- Prefer concrete decisions
- Align with repository conventions
```

## `create-implementation-plan.prompt.md`

```md
Using the approved docs for this feature, produce an implementation plan as an ordered checklist.

Requirements:
- each step small and testable
- identify dependencies
- identify risky steps
- split backend/frontend/testing/documentation work
- no code yet
```

## `implement-backend-step.prompt.md`

```md
Implement only the selected backend step.

Before coding:
- list impacted files
- state assumptions
- confirm alignment with docs/requirements/<feature>/

Then implement:
- minimal complete change
- tests if appropriate
- OpenAPI/docs updates if contract changes
```

## `validate-feature.prompt.md`

```md
Validate the implementation against:
- requerimiento.md
- analisis-funcional.md
- analisis-tecnico.md

Check:
- missing scenarios
- contract mismatches
- validation gaps
- exception handling gaps
- test gaps
- documentation drift

List issues first. Do not silently modify code.
```

## `review-pr.prompt.md`

```md
Review this branch like a strict senior teammate.

Focus on:
- correctness
- clarity
- maintainability
- architecture consistency
- Mongo query quality
- test quality
- API safety

Output:
- blocking issues
- non-blocking improvements
- nice-to-have suggestions
```

---

# 6) Advanced workflow

This is the workflow I’d recommend once you already have the starter kit in place.

## Phase A — Intake

Start from Jira, not from code.

Flow:

1. Read Jira ticket
2. Pull related Confluence standards / business notes
3. Ask Copilot to summarize requirement
4. Ask Copilot to identify unknowns
5. Write feature docs under `docs/requirements/...`

Best surface:

* Copilot CLI for focused structured prompting
* VS Code if you want to inspect code and docs side by side

## Phase B — Architecture alignment

Before implementation, force a design pass.

Ask Copilot:

* which existing modules should change
* whether endpoint is new or extension of existing API
* where Mongo custom logic should live
* how errors should map into current `@ControllerAdvice`

Outcome:

* a reviewed `analisis-tecnico.md`
* a reviewed `plan-implementacion.md`

## Phase C — Slice the work

Do not ask Copilot to “build the feature.”

Instead split into slices such as:

1. DTOs and OpenAPI contract
2. entity / model changes
3. repository / MongoTemplate logic
4. service logic
5. controller
6. exception mapping
7. tests
8. docs cleanup

This is where Copilot becomes much more reliable: smaller bounded tasks reduce drift and overreach.

## Phase D — Implement with specialist agents

Use role switching intentionally.

Suggested usage:

* architect agent → analysis/docs
* backend-spring agent → controller/service/repository changes
* mongo agent → aggregation/query work
* tester agent → tests and edge cases
* reviewer agent → pre-PR review

This works better than one all-purpose conversation because each step has a sharper operating mode.

## Phase E — Validation loop

After each slice:

1. ask for implementation review
2. ask for requirement validation
3. run tests/build yourself
4. fix only the surfaced issues
5. commit

Good loop:

```text
analyze -> implement one slice -> validate -> fix -> commit
```

Not:

```text
analyze -> implement everything -> hope
```

## Phase F — PR preparation

Before opening the PR:

Ask Copilot to produce:

* concise PR summary
* changed modules list
* risks / migration notes
* testing performed
* points reviewers should focus on

Then run reviewer agent against the diff.

---

# 7) Best division of labor: CLI vs VS Code

Based on current docs and the customization model, this is the best practical split:

## Use Copilot CLI for

* requirement analysis
* design docs generation
* implementation planning
* agent-driven focused conversations
* structured validation prompts
* targeted refactoring conversations

Copilot CLI explicitly supports custom instructions and custom agents, which makes it strong for role-based workflows. ([GitHub Docs][6])

## Use VS Code for

* prompt files
* path-aware implementation
* reading/editing code while chatting
* multi-file review
* frontend/UI work
* applying small iterative patches after inspection

VS Code supports prompt files and path-specific custom instructions, which makes it ideal for reusable workflows and repo-aware implementation sessions. ([Visual Studio Code][7])

My practical recommendation for you:

* **CLI = planning and orchestration**
* **VS Code = editing and refinement**

---

# 8) A concrete workflow for your exact stack

For a Spring Boot + Angular + MongoTemplate feature:

## Step 1

Use `architect.agent.md`

Prompt:

```text
Analyze this Jira ticket and related repository context.
Generate docs under docs/requirements/014-my-feature/.
Do not implement code.
```

## Step 2

Review docs manually.

Adjust:

* endpoint names
* DTO shape
* persistence strategy
* error cases
* backward compatibility decisions

## Step 3

Use `backend-spring.agent.md`

Prompt:

```text
Using docs/requirements/014-my-feature/, implement only step 1:
create backend DTOs and OpenAPI contract updates.
List impacted files before coding.
```

## Step 4

Use `mongo.agent.md`

Prompt:

```text
Implement only the repository/custom MongoTemplate logic for this feature.
Prefer readable programmatic aggregation/query code.
Explain assumptions about nulls, arrays, and indexes.
```

## Step 5

Use `tester.agent.md`

Prompt:

```text
Create or update tests for the backend changes.
Prefer parameterized tests when useful.
Use clear given-when-then sections.
```

## Step 6

Use `reviewer.agent.md`

Prompt:

```text
Review the current branch as a strict senior reviewer.
List blockers first, then non-blocking improvements.
```

## Step 7

Open PR with Copilot-generated summary.

---

# 9) Rules that make Copilot actually useful

These matter more than any fancy agent setup.

## Rule 1

Never start with “implement the whole feature.”

## Rule 2

Always create a written requirement and plan first.

## Rule 3

Give Copilot a single source of truth:
`docs/requirements/<feature>/`

## Rule 4

One step, one agent, one outcome.

## Rule 5

Ask for review before asking for more code.

## Rule 6

Keep your instruction files short enough to be obeyed.

A 2–4 page instruction set is usually better than a giant manifesto.

## Rule 7

Move examples/checklists into skills.

That keeps the global instructions clean and increases reuse.

---

# 10) What I would create first this week

If you want the fastest path to value, create only these first:

```text
.github/copilot-instructions.md
.github/instructions/backend.instructions.md
.github/instructions/tests.instructions.md
.github/agents/architect.agent.md
.github/agents/backend-spring.agent.md
.github/agents/reviewer.agent.md
.github/prompts/generate-feature-docs.prompt.md
.github/prompts/create-implementation-plan.prompt.md
.github/prompts/review-pr.prompt.md
.github/skills/spring-boot-standards/SKILL.md
.github/skills/mongo-patterns/SKILL.md
```

That is enough to get a serious upgrade without overengineering the setup.

---

# 11) Recommended maturity roadmap

## v1

* global instructions
* backend/test instructions
* architect + backend + reviewer agents
* 3 prompt files
* 2 skills

## v2

* frontend + mongodb + docs instructions
* tester + mongo + angular agents
* full docs/requirements workflow
* PR summary / validation prompt files
* skill examples and checklists

## v3

* org-wide shared instruction baseline
* team-standard reusable skills
* repo templates for new services
* standardized requirement folders per ticket

---

# 12) Bottom line

The best Copilot flow is not “ask smarter prompts.”

It is:

**requirements → docs → plan → tiny implementation slices → validation → review**

And the best customization model is:

**instructions for always-on behavior, agents for roles, skills for reusable expertise, prompt files for repeatable workflows.** ([GitHub Docs][8])


[1]: https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot?utm_source=chatgpt.com "Adding repository custom instructions for GitHub Copilot"
[2]: https://docs.github.com/en/copilot/reference/custom-instructions-support?utm_source=chatgpt.com "Support for different types of custom instructions"
[3]: https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli?utm_source=chatgpt.com "Creating and using custom agents for GitHub Copilot CLI"
[4]: https://code.visualstudio.com/docs/copilot/customization/prompt-files?utm_source=chatgpt.com "Use prompt files in VS Code"
[5]: https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills?utm_source=chatgpt.com "Adding agent skills for GitHub Copilot CLI"
[6]: https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions?utm_source=chatgpt.com "Adding custom instructions for GitHub Copilot CLI"
[7]: https://code.visualstudio.com/docs/copilot/customization/custom-instructions?utm_source=chatgpt.com "Use custom instructions in VS Code"
[8]: https://docs.github.com/en/copilot/reference/customization-cheat-sheet?utm_source=chatgpt.com "Copilot customization cheat sheet"
