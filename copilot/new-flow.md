Yes — a 4-stage workflow is the right way to use Copilot CLI for non-trivial tickets, and for your case I would make it **document-first, step-by-step, and validation-driven**.

A few current-tooling points matter before defining the flow: repository custom instructions are automatically included by Copilot in that repo, skills can be loaded for specialized tasks and work with Copilot CLI, and custom agents can be invoked from Copilot CLI. By contrast, **prompt files are currently an IDE feature in public preview**, not the core reusable mechanism for Copilot CLI, so for CLI-first usage you should rely mainly on **instructions + agents + skills + markdown artifacts in the repo**. ([GitHub Docs][1])

## Recommended model

Use the workflow like this:

* **Stage 1:** turn raw business input into a Jira-ready requirement document.
* **Stage 2:** turn that requirement into technical analysis, risk notes, implementation slices, and execution prompts.
* **Stage 3:** implement **one slice at a time**, including unit tests in the same slice.
* **Stage 4:** validate the implementation against requirement + design + runtime behavior, and record evidence.

That means Stage 4 should absolutely exist as its own explicit step. It should not be “just run tests.” It should include requirement traceability, functional validation, negative-path validation, and API verification artifacts such as curl or Bruno/Postman requests when the change exposes or changes endpoints. That is consistent with GitHub’s guidance to place build, test, and validation expectations into repository instructions so Copilot can repeatedly follow them. ([GitHub Docs][1])

## Best repo structure

For Copilot CLI, I’d standardize on a per-ticket working folder like this:

```text
docs/
  workflow/
    templates/
      requirement-definition.template.md
      functional-analysis.template.md
      technical-analysis.template.md
      risk-analysis.template.md
      implementation-plan.template.md
      validation-report.template.md

  tickets/
    TICKET-123-my-feature/
      01-requirement-definition.md
      02-functional-analysis.md
      03-technical-analysis.md
      04-risk-analysis.md
      05-implementation-plan.md
      06-implementation-slices.md
      07-validation-report.md
      prompts/
        stage-1-requirement-agent-input.md
        stage-2-analysis-agent-input.md
        implement-slice-01.md
        implement-slice-02.md
        validate-ticket.md
```

This works well because Copilot CLI can operate over repo files naturally, and your ticket folder becomes the single source of truth for the whole implementation path. That aligns with the current Copilot customization model: keep reusable guidance in instructions/skills/agents, and keep task-specific artifacts in repo files. ([GitHub Docs][1])

## The 4 stages in practice

## Stage 1 — Requirement definition

### Goal

Turn raw business/process input into a requirement that is ready to become a Jira ticket.

### Output file

```text
docs/tickets/TICKET-123-my-feature/01-requirement-definition.md
```

### What this file should contain

Use a structure like this:

```md
# Requirement Definition

## Title
Short business-facing title.

## Background
Why this change is needed.

## Problem Statement
What is broken, missing, or inefficient today.

## Business Goal
What outcome the business wants.

## Scope
What is included.

## Out of Scope
What is not included.

## Actors
Who uses or triggers this behavior.

## Business Rules
Rules, constraints, validations.

## Main Flow
Step-by-step happy path.

## Alternate / Error Flows
Expected non-happy cases.

## Acceptance Criteria
- AC1 ...
- AC2 ...
- AC3 ...

## Data / Inputs / Outputs
Input fields, source systems, output behavior.

## Open Questions
Unknowns that block implementation or design.

## Jira Ticket Draft
### Summary
...
### Description
...
### Acceptance Criteria
...
```

### How to use Copilot

Create a **custom agent** for requirement shaping, because this is a repeatable role-based task. The agent should be told to:

* convert raw business input into structured requirement markdown,
* identify ambiguity,
* avoid technical design too early,
* produce Jira-ready text.

That is a good fit for custom agents, which are designed to encode expertise, tooling, and response patterns for a task. ([GitHub Docs][2])

### Example CLI usage pattern

In Copilot CLI interactive mode, use your requirement agent and give it:

* the raw business process description,
* any sample payloads,
* current behavior,
* expected behavior,
* constraints.

Then ask it to write or update:

```text
docs/tickets/TICKET-123-my-feature/01-requirement-definition.md
```

### Important rule

At Stage 1, do **not** ask Copilot to implement anything yet. Only define the requirement. This separation reduces drift between “what we want” and “how we code it.”

---

## Stage 2 — Analysis, design, risk, slicing

### Goal

Transform the requirement into a safe, incremental implementation plan.

### Output files

```text
02-functional-analysis.md
03-technical-analysis.md
04-risk-analysis.md
05-implementation-plan.md
06-implementation-slices.md
```

### What each file should do

#### `02-functional-analysis.md`

What system behavior changes from a user/business perspective.

Suggested sections:

```md
# Functional Analysis
## Requirement Summary
## Current Behavior
## Target Behavior
## Impacted User Journeys
## Edge Cases
## Assumptions
## Open Questions
```

#### `03-technical-analysis.md`

Map the requirement to code and architecture.

Suggested sections:

```md
# Technical Analysis
## Affected Modules
## Affected Endpoints
## Affected DTOs / Entities
## Database / Persistence Impact
## External Integrations
## Validation / Error Handling Impact
## Backward Compatibility
## Observability / Logging Impact
## Security Considerations
```

#### `04-risk-analysis.md`

Make risk explicit before coding.

Suggested sections:

```md
# Risk Analysis
## Functional Risks
## Technical Risks
## Regression Risks
## Data Risks
## Performance Risks
## Testability Risks
## Mitigations
```

#### `05-implementation-plan.md`

High-level execution order.

Suggested sections:

```md
# Implementation Plan
## Approach
## Preconditions
## Ordered Work Items
1. ...
2. ...
3. ...

## Definition of Done
```

#### `06-implementation-slices.md`

This is the key file for Stage 3.

Suggested structure:

```md
# Implementation Slices

## Slice 01 - API contract / DTO preparation
### Goal
### Files to touch
### Runtime changes
### Unit tests
### Done when

## Slice 02 - Service logic
...

## Slice 03 - Repository / persistence
...

## Slice 04 - Error handling / controller advice
...

## Slice 05 - Endpoint integration and docs
...
```

### Why slicing matters

The most effective Copilot CLI flow is not “implement the entire ticket.” It is “implement slice 01 only,” then review, then “implement slice 02 only,” and so on. This matches good agentic usage: smaller bounded tasks produce more reliable edits and easier review. GitHub’s CLI and custom-instructions guidance both support giving Copilot build/test/validate expectations so it can work incrementally and safely. ([GitHub Docs][3])

## Stage 3 — Implementation, one slice at a time

### Goal

Implement one slice, with production code and unit tests together.

### Inputs

* `01-requirement-definition.md`
* `03-technical-analysis.md`
* `05-implementation-plan.md`
* `06-implementation-slices.md`
* current codebase
* repository instructions

### Iteration loop for each slice

For each slice, use this exact rhythm:

1. Ask Copilot to implement **only one slice**.
2. Ask it to add or update **unit tests for that slice**.
3. Review the diff yourself.
4. Run local checks.
5. Fix issues before moving to the next slice.
6. Update `06-implementation-slices.md` to mark the slice done and note deviations.

### Per-slice prompt shape

For CLI, keep a markdown file like:

```md
# Implement Slice 02

Use the ticket context from:
- docs/tickets/TICKET-123-my-feature/01-requirement-definition.md
- docs/tickets/TICKET-123-my-feature/03-technical-analysis.md
- docs/tickets/TICKET-123-my-feature/06-implementation-slices.md

Implement only:
## Slice 02 - Service logic

Rules:
- Do not implement future slices
- Keep changes minimal
- Add/update unit tests for the changed service behavior
- Reuse existing patterns in the project
- If something is ambiguous, document assumptions in the slice notes
- If endpoint behavior changes, keep API docs aligned
- If endpoint is created/updated, also create/update Bruno request files
```

### What to ask Copilot to return

Tell it to:

* edit code,
* explain files changed,
* explain why those files changed,
* list tests added/updated,
* list anything deferred to later slices.

### Why unit tests belong in the same slice

Because otherwise slice drift appears: runtime code advances, tests lag behind, and the next slice starts on unstable ground. Copilot is much more reliable when the prompt explicitly couples behavior change with test change in the same bounded task. Repository instructions are a strong place to enforce this repeatedly. ([GitHub Docs][1])

## Stage 4 — Validation

Yes: you should have a **dedicated validation prompt and output file**.

Validation should check:

1. **Requirement conformance** — did we implement what Stage 1 asked for?
2. **Design conformance** — did we respect Stage 2 decisions and risks?
3. **Runtime correctness** — do unit/integration/API tests pass?
4. **Behavior evidence** — can we prove the endpoint or flow works?
5. **Regression confidence** — what existing behavior was re-verified?

### Output file

```text
docs/tickets/TICKET-123-my-feature/07-validation-report.md
```

### Suggested structure

```md
# Validation Report

## Scope Validated
What ticket / slices / files were validated.

## Traceability Matrix
| Acceptance Criteria | Evidence | Status |
|---|---|---|
| AC1 | unit test X, curl Y | PASS |
| AC2 | service test Z | PASS |

## Technical Checks
- Build
- Unit tests
- Static analysis
- Formatting/linting

## Integration / API Checks
- curl examples
- Bruno/Postman collection updates
- Sample request/response evidence

## Negative Cases Checked
- Validation errors
- Not found
- Conflict / duplicates
- Unauthorized / forbidden if applicable

## Deviations
Anything implemented differently than planned.

## Remaining Risks
Known gaps that still exist.

## Final Status
PASS / PASS WITH NOTES / FAIL
```

### Should you run integration tests?

For endpoint work: **yes, ideally**. At minimum:

* unit tests,
* one or more API-level checks,
* example curl commands or Bruno requests,
* negative-path verification.

If the repo already supports Spring Boot integration tests, include focused ones for critical flows. If not, at least create executable API checks and document them in validation. This is especially valuable for REST endpoints because it proves request mapping, serialization, validation, and error handling together.

### curl / Bruno / Postman?

For your workflow, I’d standardize on:

* **curl examples in the validation report** for portability,
* **Bruno requests in the repo** for repeatable team usage.

That gives you both “fast evidence” and “team artifact.” If endpoint creation/update is common in your work, put that into repository instructions or a dedicated skill so Copilot consistently remembers to update the Bruno folder. That matches the use case of repository instructions for project-wide rules and skills for specialized tasks. ([GitHub Docs][1])

## The best Copilot customization setup for this workflow

For CLI-first usage, I’d split responsibilities like this:

### 1. Repository instructions

Use for stable, always-on rules.

Good examples:

* build/test commands,
* coding style,
* “implement one slice at a time,”
* “always add/update unit tests with code changes,”
* “for REST endpoint changes, update Bruno requests,”
* “document validation evidence in `07-validation-report.md`.”

This is exactly what repository custom instructions are for. ([GitHub Docs][1])

### 2. Custom agents

Use for role-driven tasks.

Recommended agents:

* `requirement-author`
* `solution-analyzer`
* `slice-implementer`
* `validator`

This fits the documented purpose of custom agents in Copilot CLI. ([GitHub Docs][2])

### 3. Skills

Use for specialized reusable capabilities.

Recommended skills:

* `jira-ticket-formatting`
* `api-bruno-sync`
* `spring-boot-testing`
* `validation-evidence`

Skills are the right place for instructions plus resources/scripts that should activate for specialized tasks. ([GitHub Docs][4])

### 4. Ticket-local markdown files

Use for this ticket’s artifacts and prompts.

That gives you a clean separation:

* global/repo behavior in instructions,
* reusable expertise in agents/skills,
* ticket-specific state in docs.

## A concrete workflow you can actually follow

Here is the operational path I recommend.

### Step A — Create ticket workspace

Create:

```text
docs/tickets/TICKET-123-my-feature/
```

### Step B — Stage 1

Use `requirement-author` agent on raw business input.

Generate:

```text
01-requirement-definition.md
```

Review it manually and fix ambiguity before continuing.

### Step C — Stage 2

Use `solution-analyzer` agent on the requirement.

Generate:

```text
02-functional-analysis.md
03-technical-analysis.md
04-risk-analysis.md
05-implementation-plan.md
06-implementation-slices.md
```

Review especially:

* open questions,
* risk assumptions,
* slice order.

### Step D — Stage 3, repeated

For each slice in `06-implementation-slices.md`:

* run `slice-implementer`,
* implement only that slice,
* add/update unit tests,
* run build/tests,
* review diff,
* update slice status in the doc,
* commit.

A good commit rhythm is one commit per slice.

### Step E — Stage 4

Use `validator` agent.

Generate:

```text
07-validation-report.md
```

Have it verify:

* acceptance criteria mapping,
* tests,
* API checks,
* unresolved deviations.

## Minimal file set if you want to keep it lean

If you want the smallest version that still works well, use only these:

```text
01-requirement-definition.md
03-technical-analysis.md
05-implementation-plan.md
06-implementation-slices.md
07-validation-report.md
```

That is enough for a practical workflow.

## My recommendation for your exact question

Yes, your 4 stages are correct. I would refine them as:

1. **Requirement Definition**

   * Output: `01-requirement-definition.md`

2. **Analysis and Planning**

   * Outputs: `02-functional-analysis.md`, `03-technical-analysis.md`, `04-risk-analysis.md`, `05-implementation-plan.md`, `06-implementation-slices.md`

3. **Implementation by Slice**

   * Output: code + tests + slice status updates
   * Rule: always one slice at a time

4. **Validation and Evidence**

   * Output: `07-validation-report.md`
   * Include requirement traceability, tests, curl/Bruno evidence, negative cases, and deviations

And yes, for REST endpoints, I strongly recommend:

* unit tests,
* integration or API-level validation where feasible,
* curl examples,
* Bruno/Postman request artifacts,
* explicit mapping from acceptance criteria to evidence.

That gives you a workflow that is predictable for humans and easier for Copilot to execute consistently. ([GitHub Docs][1])

In the next step, I can turn this into a **ready-to-paste starter kit** with actual contents for:

* `.github/copilot-instructions.md`
* 4 custom agent files
* 3–4 skills
* markdown templates for all 7 output files

[1]: https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot?utm_source=chatgpt.com "Adding repository custom instructions for GitHub Copilot"
[2]: https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli?utm_source=chatgpt.com "Creating and using custom agents for GitHub Copilot CLI"
[3]: https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli?utm_source=chatgpt.com "Using GitHub Copilot CLI"
[4]: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/add-skills?utm_source=chatgpt.com "Adding agent skills for GitHub Copilot"

Place these in `.github/agents/`. Copilot CLI custom agents are defined as Markdown files with the `.agent.md` extension, and repository-level agents live in `.github/agents/`. ([GitHub Docs][1])

### `.github/agents/requirement-author.agent.md`

```md
---
name: Requirement Author
description: Turns raw business input into a structured requirement-definition.md ready to become a Jira ticket.
tools: ['codebase', 'editFiles', 'search', 'usages']
---

You are a senior business analyst and technical writer.

Your role is to transform raw feature requests, business process descriptions, notes, tickets, chat messages, and API contract fragments into a clean and structured requirement document.

## Primary goal

Produce or update:

`docs/tickets/<ticket>/01-requirement-definition.md`

If the `<ticket>` folder is not provided in the prompt, infer a reasonable slug from the feature title and state the chosen folder name clearly.

## What you must do

1. Read the user input carefully.
2. Identify:
   - the business need
   - the current problem
   - expected target behavior
   - scope
   - out-of-scope items
   - business rules
   - happy path
   - alternate and error flows
   - assumptions
   - open questions
   - acceptance criteria
3. Convert ambiguous or unstructured input into clear requirement language.
4. Separate facts from assumptions.
5. If information is missing, do not block. Add an `Open Questions` section.
6. Do not design the implementation yet.
7. Do not write production code.
8. Be concise, structured, and Jira-friendly.

## Output format

Create the file with exactly this structure:

# Requirement Definition

## Title
A short business-facing title.

## Background
Why this change is needed.

## Problem Statement
What is wrong, missing, or inefficient today.

## Business Goal
What outcome is expected.

## Scope
Bulleted list.

## Out of Scope
Bulleted list.

## Actors
Who triggers or uses this behavior.

## Business Rules
Bulleted list of rules and constraints.

## Main Flow
Numbered steps.

## Alternate / Error Flows
Bulleted or numbered list.

## Data / Inputs / Outputs
Describe relevant input fields, source systems, and expected outputs.

## Acceptance Criteria
Numbered list of testable acceptance criteria.

## Assumptions
Bulleted list.

## Open Questions
Bulleted list.

## Jira Ticket Draft

### Summary
One-line summary.

### Description
A compact Jira-ready description.

### Acceptance Criteria
Repeat the acceptance criteria in Jira-friendly form.

## Writing rules

- Prefer precise statements over generic prose.
- Acceptance criteria must be testable.
- Avoid implementation details unless the user explicitly included them as constraints.
- Avoid saying "the system should" repeatedly; vary wording naturally.
- Keep the document readable by both business and engineering.
- If the user gives an API contract, reflect it as requirement behavior, not as code design.
- Use Markdown only.

## Response behavior

- If asked to create the file, write the file directly.
- If asked to refine an existing requirement-definition.md, preserve valid existing content and improve weak sections.
- At the end of your response, summarize:
  - what you created or changed
  - assumptions introduced
  - open questions that remain
```

### `.github/agents/solution-analyzer.agent.md`

```md
---
name: Solution Analyzer
description: Reads the requirement and creates analysis, risks, implementation plan, and implementation slices for incremental delivery.
tools: ['codebase', 'editFiles', 'search', 'usages']
---

You are a senior software architect, systems analyst, and implementation planner.

Your role is to read an approved requirement and transform it into implementation analysis and a step-by-step delivery plan.

## Primary goal

Use:

`docs/tickets/<ticket>/01-requirement-definition.md`

Produce or update these files:

- `docs/tickets/<ticket>/02-functional-analysis.md`
- `docs/tickets/<ticket>/03-technical-analysis.md`
- `docs/tickets/<ticket>/04-risk-analysis.md`
- `docs/tickets/<ticket>/05-implementation-plan.md`
- `docs/tickets/<ticket>/06-implementation-slices.md`

## What you must do

1. Read the requirement file first.
2. Inspect the repository structure and relevant code.
3. Identify impacted layers, modules, endpoints, DTOs, entities, persistence, integrations, validation rules, observability, and security considerations.
4. Document both functional and technical impact.
5. Identify risks and mitigations.
6. Build an implementation plan in a safe execution order.
7. Split the work into small implementation slices whenever possible.
8. Each slice must be independently implementable and reviewable.
9. Do not implement code in this stage.
10. Do not skip unclear areas; document them as assumptions or open questions.

## File requirements

### 02-functional-analysis.md

Use this structure:

# Functional Analysis

## Requirement Summary
## Current Behavior
## Target Behavior
## Affected User Journeys
## Edge Cases
## Assumptions
## Open Questions

### 03-technical-analysis.md

Use this structure:

# Technical Analysis

## Relevant Existing Components
## Affected Modules and Packages
## Affected Endpoints / APIs
## DTO / Entity / Model Impact
## Persistence Impact
## External Service / Messaging / Integration Impact
## Validation and Error Handling Impact
## Security Impact
## Observability / Logging / Metrics Impact
## Backward Compatibility Notes
## Recommended Technical Approach
## Open Questions

### 04-risk-analysis.md

Use this structure:

# Risk Analysis

## Functional Risks
## Technical Risks
## Regression Risks
## Data Risks
## Performance Risks
## Testability Risks
## Mitigations
## Residual Risks

### 05-implementation-plan.md

Use this structure:

# Implementation Plan

## Strategy
## Preconditions
## Ordered Work Items
## Definition of Done
## Suggested Commit Strategy

### 06-implementation-slices.md

Use this structure:

# Implementation Slices

For each slice use:

## Slice NN - <short title>

### Goal
### Repository areas to change
### Runtime changes
### Tests to add or update
### Risks / notes
### Done when

## Slicing rules

- Prefer vertical slices over layer-only slices when practical.
- Keep each slice small enough to review comfortably.
- A slice should usually fit in one focused implementation session.
- A slice must include its related unit test work.
- Do not combine unrelated changes.
- If API changes are involved, include contract/docs/request artifact updates in the relevant slice.
- If a Bruno request should be created or updated, mention it explicitly in the slice.

## Writing rules

- Be concrete about impacted files or packages when you can infer them safely.
- Do not invent architecture that does not match the repository.
- Prefer extending existing patterns over introducing new patterns.
- Make tradeoffs explicit.
- Be honest when something is uncertain.

## Response behavior

At the end of your response, summarize:
- the main impacted areas
- the highest risks
- the proposed slice order
- any blocking open questions
```

### `.github/agents/slice-implementer.agent.md`

```md
---
name: Slice Implementer
description: Implements exactly one planned slice at a time, including runtime code and unit tests.
tools: ['codebase', 'editFiles', 'search', 'usages', 'runCommands']
---

You are a senior software engineer focused on safe, incremental implementation.

Your role is to implement exactly one slice from the ticket plan, and to add or update the unit tests needed for that slice.

## Primary goal

Read these ticket files first:

- `docs/tickets/<ticket>/01-requirement-definition.md`
- `docs/tickets/<ticket>/03-technical-analysis.md`
- `docs/tickets/<ticket>/05-implementation-plan.md`
- `docs/tickets/<ticket>/06-implementation-slices.md`

Then implement only the slice explicitly requested by the user.

## Strict execution rules

1. Implement one slice at a time.
2. Do not start future slices.
3. Keep the change set minimal and focused.
4. Add or update unit tests in the same slice.
5. Reuse existing project patterns and conventions.
6. If the slice changes or creates an API endpoint:
   - update API documentation if present
   - create or update the relevant Bruno request artifacts if the repository uses Bruno
7. Preserve backward compatibility unless the requirement explicitly changes behavior.
8. If something is ambiguous, choose the safest minimal assumption and state it.
9. Do not perform broad refactors unless they are necessary for the slice.
10. Do not silently change unrelated code.

## Implementation workflow

For the requested slice:

1. Identify the target files and surrounding code.
2. Explain the intended implementation before editing when useful.
3. Make the code changes.
4. Add or update unit tests.
5. If appropriate, run focused commands to validate compilation or tests.
6. Summarize what was changed and what remains for later slices.

## Test rules

- Always include unit tests for the behavior introduced or changed in the slice.
- Follow existing project test patterns.
- Cover happy path plus relevant negative paths for the slice.
- Keep tests focused on the slice scope.
- Do not claim coverage you did not add.

## Documentation update rules

If useful, update:
- `docs/tickets/<ticket>/06-implementation-slices.md`

For the implemented slice, append a short note under the slice such as:
- implementation notes
- deviations from plan
- assumptions taken

Do not mark future slices as completed.

## Response behavior

At the end of your response, include:

- Implemented slice
- Files changed
- Tests added or updated
- Commands run and results
- Assumptions made
- Deferred items for later slices

## Style

- Be precise and practical.
- Prefer small diffs.
- Prefer correctness and readability over cleverness.
- Never say a slice is done if tests clearly fail.
```

### `.github/agents/validator.agent.md`

```md
---
name: Validator
description: Validates the implemented slices against the requirement, plan, and observed behavior, and writes a validation report.
tools: ['codebase', 'editFiles', 'search', 'usages', 'runCommands']
---

You are a senior QA-oriented engineer and technical reviewer.

Your role is to validate that the implementation matches:
1. the requirement
2. the technical plan
3. the intended runtime behavior

## Primary goal

Read these files first:

- `docs/tickets/<ticket>/01-requirement-definition.md`
- `docs/tickets/<ticket>/03-technical-analysis.md`
- `docs/tickets/<ticket>/05-implementation-plan.md`
- `docs/tickets/<ticket>/06-implementation-slices.md`

Produce or update:

`docs/tickets/<ticket>/07-validation-report.md`

## Validation responsibilities

You must validate the implementation from several angles:

### 1. Requirement conformance
Check whether the code satisfies the stated requirement and acceptance criteria.

### 2. Plan conformance
Check whether the implementation broadly follows the intended design and slice plan.
If it deviates, document the deviation rather than hiding it.

### 3. Code and test evidence
Inspect relevant code and tests.
Run focused validation commands when possible.

### 4. Runtime/API behavior
For endpoint changes, verify request/response expectations as much as possible.
If runnable API validation is not possible, provide exact curl or Bruno examples that should be executed.

### 5. Negative-path validation
Check whether validation errors, not-found cases, and other expected failures are handled appropriately where relevant.

## Output format

Write the report with this structure:

# Validation Report

## Scope Validated
What ticket, slices, and main areas were validated.

## Acceptance Criteria Traceability
For each acceptance criterion:
- criterion
- evidence
- status: PASS / PARTIAL / FAIL

## Planned vs Implemented
Summarize whether the implementation matches the intended plan and note deviations.

## Code Review Findings
Observations about correctness, maintainability, error handling, and test quality.

## Technical Checks Performed
List commands executed and their outcomes.

## API / Integration Validation
Include curl examples, request/response expectations, or Bruno/Postman notes when relevant.

## Negative Cases Checked
List failures and error scenarios that were validated.

## Risks or Gaps Remaining
Anything still uncertain or unverified.

## Final Status
One of:
- PASS
- PASS WITH NOTES
- FAIL

## Validator rules

- Be evidence-based.
- Do not mark PASS if key acceptance criteria are unverified.
- Be explicit about what was actually validated versus what was only inferred.
- If you cannot run something, say so and provide the exact command or request to run.
- Prefer traceability over verbosity.
- Do not implement new production code in this stage unless the user explicitly asks for fixes after validation.

## Response behavior

At the end of your response, summarize:
- final status
- failed or partial criteria
- recommended next actions
```

These four agents match the current GitHub model for custom agent profiles in Markdown and are appropriate for Copilot CLI repository-level setup. ([GitHub Docs][2])

In the next step, I’d suggest creating the matching `.github/copilot-instructions.md` so these agents behave consistently across all tickets.

[1]: https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli?utm_source=chatgpt.com "Creating and using custom agents for GitHub Copilot CLI"
[2]: https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-custom-agents?utm_source=chatgpt.com "About custom agents"

Use this as your repository-level `.github/copilot-instructions.md`:

```md
# Copilot Repository Instructions

This repository uses a document-first, slice-based workflow for ticket delivery.

Copilot must follow the staged workflow below and keep all generated artifacts aligned with it.

---

## 1. Core working model

For non-trivial work, always think in these stages:

1. Requirement definition
2. Analysis and planning
3. Implementation by slice
4. Validation

Do not jump directly from a vague request to a large implementation.

When the request is incomplete or ambiguous:
- first structure the requirement
- then analyze and plan
- then implement one slice at a time
- then validate against the requirement and plan

---

## 2. Ticket workspace convention

For each ticket or feature, use a workspace under:

`docs/tickets/<ticket>/`

Expected files:

- `01-requirement-definition.md`
- `02-functional-analysis.md`
- `03-technical-analysis.md`
- `04-risk-analysis.md`
- `05-implementation-plan.md`
- `06-implementation-slices.md`
- `07-validation-report.md`

If the user gives a feature name but no ticket id, infer a safe folder name and state it clearly.

Do not scatter ticket documentation across random locations.

---

## 3. Stage-specific behavior

### Stage 1 - Requirement definition

If the input is raw business context, a Jira description, chat notes, API notes, or a rough problem statement:
- do not implement code yet
- first create or refine `01-requirement-definition.md`

The requirement document should clarify:
- background
- problem
- goal
- scope
- out of scope
- business rules
- main flow
- alternate/error flows
- inputs/outputs
- acceptance criteria
- assumptions
- open questions

Acceptance criteria must be testable and concrete.

Avoid premature technical design in this stage unless the user explicitly asks for technical constraints to be captured.

### Stage 2 - Analysis and planning

Before implementing non-trivial changes, create or update:
- `02-functional-analysis.md`
- `03-technical-analysis.md`
- `04-risk-analysis.md`
- `05-implementation-plan.md`
- `06-implementation-slices.md`

In this stage:
- inspect the existing codebase first
- prefer existing repository patterns over inventing new structures
- identify impacted packages, modules, DTOs, endpoints, persistence, validation, logging, security, external integrations, and tests
- document risks explicitly
- split work into small slices whenever possible

Do not implement production code in Stage 2.

### Stage 3 - Implementation by slice

Implementation must be incremental.

Always:
- implement exactly one slice at a time
- keep the diff focused
- add or update unit tests in the same slice
- avoid broad refactors unless required for the slice
- preserve backward compatibility unless the requirement explicitly changes it

Before editing code for a planned ticket, read:
- `01-requirement-definition.md`
- `03-technical-analysis.md`
- `05-implementation-plan.md`
- `06-implementation-slices.md`

When relevant, update `06-implementation-slices.md` with implementation notes, assumptions, or deviations for the slice that was implemented.

Do not silently implement future slices.

### Stage 4 - Validation

After implementation, create or update:
- `07-validation-report.md`

Validation must check:
- requirement conformance
- acceptance criteria traceability
- alignment with the technical plan
- test evidence
- runtime or API behavior where relevant
- negative-path behavior where relevant
- remaining risks or gaps

Do not mark work as fully done without evidence.

---

## 4. Slice policy

Prefer small, reviewable slices.

A slice should be:
- independently understandable
- testable
- narrow in scope
- safe to review and commit on its own

Prefer vertical slices over large layer-only waves when practical.

Each slice should describe:
- goal
- repository areas to change
- runtime changes
- tests to add or update
- risks/notes
- done-when criteria

Each implementation slice should usually result in:
- one focused code change
- matching unit test updates
- a small, reviewable commit

---

## 5. Code change rules

When modifying code:
- follow existing project patterns first
- keep changes minimal and targeted
- avoid unrelated cleanup
- do not rename or move files without reason
- do not introduce a new architectural pattern unless clearly justified
- do not invent missing business rules; document assumptions instead

If something is unclear:
- choose the safest minimal assumption
- state it explicitly in the response and, if relevant, in the ticket docs

Never claim something is implemented if it has not actually been changed.

---

## 6. Testing expectations

Whenever runtime behavior changes, add or update tests in the same slice.

Testing guidance:
- prioritize focused unit tests
- cover happy path and relevant negative paths
- follow existing test style and conventions
- keep tests readable and targeted
- do not claim coverage you did not add

If commands can be run safely, use focused validation such as:
- project build
- targeted test classes
- module-specific test runs

If you cannot run a validation step, say so clearly.

---

## 7. API and endpoint rules

If a slice creates or changes an API endpoint:
- update the API contract or endpoint documentation if the project uses it
- keep request/response behavior aligned with the requirement
- review validation and error handling behavior
- include negative-path coverage where relevant

If the repository uses Bruno for API requests:
- create or update the corresponding Bruno request
- keep environment values or collection references aligned with the endpoint change

For API validation, prefer documenting:
- exact curl examples
- expected request body / params
- expected response
- expected error cases when relevant

---

## 8. Validation and traceability rules

For validation, be evidence-based.

The validation report should map acceptance criteria to evidence such as:
- unit tests
- integration tests
- curl checks
- Bruno requests
- code paths reviewed
- observed runtime behavior

Allowed statuses in validation:
- PASS
- PASS WITH NOTES
- FAIL

Do not mark PASS if important acceptance criteria are still unverified.

Be explicit about:
- what was actually tested
- what was inferred from code review
- what remains unverified

---

## 9. Documentation behavior

When generating documentation:
- use Markdown
- prefer concise and structured writing
- keep content practical and implementation-oriented
- avoid filler prose
- preserve useful existing content when refining a file

When updating an existing ticket workspace:
- keep filenames stable
- extend the existing artifact instead of replacing it blindly
- retain valid prior decisions unless the prompt asks to revise them

---

## 10. Agent usage guidance

This repository may use specialized agents for the workflow.

Use the matching agent when appropriate:

- `Requirement Author` for Stage 1
- `Solution Analyzer` for Stage 2
- `Slice Implementer` for Stage 3
- `Validator` for Stage 4

When acting without an explicit agent selection, still follow the same staged behavior.

---

## 11. Response style

When responding:
- be clear and practical
- explain what you changed
- list important assumptions
- call out open questions
- distinguish completed work from deferred work

For implementation responses, include:
- the slice implemented
- key files changed
- tests added or updated
- commands run and results
- assumptions or deviations

For validation responses, include:
- final status
- criteria that passed
- criteria that are partial or failed
- recommended next actions

---

## 12. What not to do

Do not:
- implement an entire complex ticket in one step unless the user explicitly wants that and the change is actually small
- skip requirement clarification for vague requests
- skip planning for non-trivial work
- bundle unrelated changes into the same slice
- mark validation as complete without evidence
- hide uncertainty
- silently ignore risks or open questions

---

## 13. Default decision rule

If in doubt:
- reduce scope
- document assumptions
- implement one smaller slice
- validate before moving on
```

This should make the 4 agents behave consistently, because it gives them a shared repo-wide contract while still letting each agent specialize in its stage.

Next, the most useful follow-up is to create the matching `.github/skills/` pack so the agents can reuse standardized behaviors like Jira formatting, implementation slicing, Bruno sync, and validation evidence.

Yes. I’d lock in this **final 5-skill set** for your workflow:

1. `ticket-context-gathering`
2. `mermaid-requirement-diagrams`
3. `implementation-slicing`
4. `spring-endpoint-with-bruno`
5. `validation-evidence`

Why this set: skills are best for **specialized, situational procedures**, while custom agents are better for **workflow roles**, and MCP is the way Copilot connects to external systems like Jira, Confluence, Jenkins, or logs tools. GitHub’s docs explicitly frame skills as folders of instructions/scripts/resources loaded when relevant, custom agents as specialized versions of Copilot for particular tasks, and MCP as the integration layer for external tools. ([GitHub Docs][1])

I would *not* make the final 5 include a generic `jira-skill` or `confluence-skill`, because that is too tool-centric. Instead, the better pattern is a workflow skill that **uses MCP-backed tools when needed**, such as gathering ticket context or collecting validation evidence. That keeps the skill focused on the job, not the product. Also, since skills work with Copilot CLI, you can use this pack directly in your CLI-first flow. ([GitHub Docs][1])

## 1. `ticket-context-gathering`

**Purpose**
Collect the business and technical context needed before writing the requirement or the analysis plan.

**Why it belongs in the final set**
This is the best place to use Jira/Confluence MCP. It helps Stage 1 and Stage 2 without forcing your `Requirement Author` or `Solution Analyzer` agents to reinvent the same retrieval steps every time.

**Trigger**
Use it when:

* the prompt references a Jira ticket, story, epic, bug, or issue
* the prompt mentions Confluence/wiki pages, design docs, or acceptance criteria
* the task begins with “analyze this ticket,” “structure this requirement,” or “plan this feature”
* linked documentation or ticket context exists outside the repo

**Inputs**

* Jira ticket key, link, or pasted ticket text
* Confluence page link, wiki title, or pasted notes
* raw business description
* optional: linked PRs, issue comments, related tasks

**Outputs**
Usually contributes to:

* `01-requirement-definition.md`
* `02-functional-analysis.md`
* `03-technical-analysis.md`

What it should output in practice:

* concise context summary
* business goal
* acceptance criteria found
* linked docs found
* unresolved ambiguities
* assumptions needed before planning

**Should it call MCP-backed tools?**
**Yes.** Strongly yes.
This is the most natural MCP-backed skill in your workflow, because MCP is exactly how Copilot integrates with external systems across Copilot surfaces, including Copilot CLI. ([GitHub Docs][2])

---

## 2. `mermaid-requirement-diagrams`

**Purpose**
Propose and generate the most useful Mermaid diagram for a requirement whenever a diagram would improve clarity.

**Why it belongs in the final set**
You explicitly want the requirement stage to propose diagrams. This is a classic skill: it is reusable, specialized, and should activate only when the requirement benefits from visualization.

**Trigger**
Use it when:

* the requirement describes a multi-step process
* there are decision branches
* multiple actors/systems interact
* there are state transitions
* the request explicitly asks for a Mermaid diagram
* the requirement would be clearer with a flow or interaction view

**Inputs**

* `01-requirement-definition.md` draft or raw business input
* optional ticket context from Jira/Confluence
* optional API or process notes

**Outputs**
Usually adds one section to `01-requirement-definition.md`, for example:

````md
## Suggested Diagram
### Mermaid - Business Flow
```mermaid
flowchart TD
...
````

```

or possibly to `02-functional-analysis.md`.

**What it should choose between**
- `flowchart` for process/decision flow
- `sequenceDiagram` for actor/system interactions
- `stateDiagram-v2` for lifecycle/status changes
- `erDiagram` only when relationships really matter
- skip the diagram if it adds no value

**Should it call MCP-backed tools?**  
**Usually no.**  
It can benefit indirectly from context gathered through Jira/Confluence, but the skill itself is mainly about diagram selection and generation, not external tool access.

---

## 3. `implementation-slicing`

**Purpose**  
Break a requirement and technical analysis into small, safe, reviewable implementation slices.

**Why it belongs in the final set**  
This is one of the most important skills for making Copilot useful in real projects. It prevents the common failure mode of “implement the entire ticket in one pass.”

**Trigger**  
Use it when:
- Stage 2 is generating `06-implementation-slices.md`
- a ticket is non-trivial
- the change affects multiple layers
- the user asks for a safe step-by-step plan
- you want one-slice-per-commit execution

**Inputs**  
- `01-requirement-definition.md`
- `03-technical-analysis.md`
- repo structure and existing code patterns
- optional architecture constraints

**Outputs**  
Primary output:
- `06-implementation-slices.md`

Secondary output:
- slice boundaries
- ordering
- dependencies
- done-when criteria
- tests per slice

**What it should enforce**
- one slice = one focused change
- runtime + unit tests together
- avoid bundling unrelated work
- vertical slices preferred when practical
- mention Bruno/API docs if endpoint changes are part of the slice

**Should it call MCP-backed tools?**  
**Usually no.**  
It mostly operates on the repo and the ticket docs. It may benefit from prior ticket context, but it does not need direct MCP access in most cases.

---

## 4. `spring-endpoint-with-bruno`

**Purpose**  
Standardize what happens whenever a Spring Boot API endpoint is created or changed.

**Why it belongs in the final set**  
This directly matches your working style: endpoint changes should also update request artifacts. It is narrow, repeatable, and high value.

**Trigger**  
Use it when:
- adding a new controller endpoint
- changing request/response shape
- changing validation or status codes
- changing a service contract exposed through REST
- updating API docs for an existing endpoint

**Inputs**  
- relevant slice from `06-implementation-slices.md`
- endpoint requirement from `01-requirement-definition.md`
- existing controller/service/DTO patterns in repo
- existing Bruno collection structure
- optional OpenAPI/Swagger conventions

**Outputs**  
Code and artifact expectations:
- controller/service/DTO changes as needed
- OpenAPI/Swagger alignment where used
- Bruno request creation or update
- environment values or notes if required
- expected status codes and example request payloads

This skill should typically support Stage 3 and feed evidence into Stage 4.

**Should it call MCP-backed tools?**  
**No, normally.**  
This is repo-local work. It does not usually need Jira/Confluence/Jenkins/logs access.

---

## 5. `validation-evidence`

**Purpose**  
Turn validation into an evidence-based activity instead of a vague “looks good.”

**Why it belongs in the final set**  
This is what gives Stage 4 real value. Without it, validation often degrades into an informal review.

**Trigger**  
Use it when:
- implementation for one or more slices is complete
- a PR is being prepared
- an endpoint or runtime behavior changed
- you need traceability from ACs to evidence
- the user asks whether the feature is ready

**Inputs**  
- `01-requirement-definition.md`
- `05-implementation-plan.md`
- `06-implementation-slices.md`
- changed code and tests
- build/test results
- optional curl examples / Bruno requests
- optional Jenkins build results
- optional logs when validating bugfixes or production-like failures

**Outputs**  
Primary output:
- `07-validation-report.md`

What it should contain:
- acceptance criteria traceability
- evidence per criterion
- commands run
- tests reviewed or executed
- API checks/curl examples
- negative-path validation
- risks or gaps remaining
- final status: PASS / PASS WITH NOTES / FAIL

**Should it call MCP-backed tools?**  
**Yes, optionally.**  
This is where Jenkins and log/Elasticsearch MCP can become very valuable:
- Jenkins for build/pipeline evidence
- log tools for bugfix validation or failure correlation
- Jira for restating ACs if needed

So this should be an MCP-capable skill, but not MCP-dependent in every run.

---

## Why this final 5 is stronger than the earlier generic pack

The reason I prefer this over a pack like `jira-requirement-format` + `unit-test-patterns` is that your workflow now explicitly includes:
- potential MCP integration with Jira/Confluence/Jenkins/logs
- a strong desire for Mermaid at requirement time
- endpoint artifact syncing with Bruno
- evidence-driven validation

So these 5 line up better with how you actually work:
- **Stage 1**: `ticket-context-gathering`, `mermaid-requirement-diagrams`
- **Stage 2**: `implementation-slicing`
- **Stage 3**: `spring-endpoint-with-bruno`
- **Stage 4**: `validation-evidence`

That division keeps agents focused on the role and skills focused on the repeatable sub-procedure, which is the cleanest design according to GitHub’s customization model. :contentReference[oaicite:3]{index=3}

## Recommended MCP usage policy for these 5

A practical rule set:

- `ticket-context-gathering` → **MCP by default when ticket/docs exist**
- `mermaid-requirement-diagrams` → **no MCP needed**
- `implementation-slicing` → **no MCP by default**
- `spring-endpoint-with-bruno` → **no MCP needed**
- `validation-evidence` → **MCP optional when build/log evidence exists**

That gives you a good balance: not every skill depends on external systems, but the ones that benefit from them are designed to use them.

## My final pick, in one line

Use these exact 5:

- `ticket-context-gathering`
- `mermaid-requirement-diagrams`
- `implementation-slicing`
- `spring-endpoint-with-bruno`
- `validation-evidence`

Next, I can draft the exact `.github/skills/` folder names and the contents of each `SKILL.md`.
::contentReference[oaicite:4]{index=4}
```

[1]: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/add-skills?utm_source=chatgpt.com "Adding agent skills for GitHub Copilot"
[2]: https://docs.github.com/en/copilot/concepts/context/mcp?utm_source=chatgpt.com "About Model Context Protocol (MCP)"

Short answer: **you don’t need to rewrite anything — but you *should* do a light alignment pass** so agents naturally leverage the new skills.

Right now:

* Your **agents are solid** (clear roles, no overlap 👍)
* Your **instructions are solid** (workflow + rules 👍)
* Your **skills will add power**, but **nothing currently tells agents *when to use them***

So things will still work… just not *optimally*.

---

# What should you change?

Think of it like this:

| Component    | Role                  | Status | Needed change                        |
| ------------ | --------------------- | ------ | ------------------------------------ |
| Instructions | Global rules          | ✅ Good | 🔧 Minor hints                       |
| Agents       | Workflow roles        | ✅ Good | 🔧 Add “use skills when appropriate” |
| Skills       | Specialized execution | 🆕 New | ✅ Will be used                       |

---

# 1. Do you need to modify the agents?

👉 **Yes — but only slightly (recommended, not mandatory)**

You don’t want to rewrite them.
You just want to **teach them to use skills opportunistically**.

## What to add (pattern)

In each agent, add something like:

```md
## Skill usage

When relevant, use available skills to improve output quality.

Examples:
- Use `ticket-context-gathering` when external ticket or documentation context exists
- Use `mermaid-requirement-diagrams` when a diagram can clarify the requirement or flow
- Use `implementation-slicing` when defining or refining slices
- Use `spring-endpoint-with-bruno` when creating or modifying API endpoints
- Use `validation-evidence` when producing validation output
```

---

## Where to add it

### Requirement Author

Add:

* ticket-context-gathering
* mermaid-requirement-diagrams

### Solution Analyzer

Add:

* ticket-context-gathering (optional)
* implementation-slicing
* mermaid (optional)

### Slice Implementer

Add:

* spring-endpoint-with-bruno

### Validator

Add:

* validation-evidence
* ticket-context-gathering (optional for AC recovery)

---

👉 Important:
**Do NOT force skills.**
Say *“when relevant”*, not *“always use”*.

---

# 2. Do you need to modify `.github/copilot-instructions.md`?

👉 **Yes — one small but important addition**

Right now your instructions define the workflow perfectly.

What’s missing is **how skills fit into it**.

## Add this section at the end

```md
---

## 14. Skill usage policy

This repository may include specialized skills under `.github/skills/`.

Guidelines:

- Use skills when they improve precision, consistency, or reuse of logic
- Do not force skill usage when not relevant
- Prefer skills for:
  - external context gathering (Jira, Confluence, logs, builds)
  - requirement visualization (Mermaid diagrams)
  - implementation slicing
  - API endpoint standardization (including Bruno requests)
  - validation evidence generation

Mapping of common scenarios:

- Requirement creation:
  - consider `ticket-context-gathering`
  - consider `mermaid-requirement-diagrams`

- Analysis and planning:
  - use `implementation-slicing`
  - optionally use `ticket-context-gathering`

- Implementation:
  - use `spring-endpoint-with-bruno` when endpoints are affected

- Validation:
  - use `validation-evidence`
  - optionally use external systems (builds/logs) if available

Skills are complementary to agents:
- agents define *what role is being performed*
- skills define *how specific tasks are executed*

Do not duplicate skill logic manually if a skill exists.
```

---

# 3. What you should NOT change

Do **not**:

* merge skills into agents ❌
* rewrite agents to depend on skills ❌
* move logic from instructions into skills ❌
* make skills mandatory ❌

Why?

Because GitHub’s model is:

* instructions → always-on baseline
* agents → orchestration/role
* skills → optional specialization

If you blur them, Copilot becomes less predictable.

---

# 4. What happens if you don’t update anything?

Everything still works, but:

* Copilot may **ignore skills more often**
* It may **re-implement logic already in skills**
* You lose consistency (especially for:

  * slicing
  * validation
  * endpoint + Bruno sync)

So:
👉 It’s not required
👉 But it’s definitely worth doing

---

# Final recommendation

Do this minimal update:

### ✅ Add a `## Skill usage` section to each agent

### ✅ Add a `## Skill usage policy` section to instructions

That’s it.

---

If you want next step, I can:
👉 generate the **exact `.github/skills/` folder structure + SKILL.md files** fully aligned with everything you built so far.
