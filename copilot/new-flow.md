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



------------------------------------


Yes — with one caveat:

You can give Copilot CLI **one command / one prompt** that asks it to go from **Jira → Requirement → Analysis → Slices**, and Copilot CLI can work through multi-step tasks autonomously in autopilot mode. Copilot CLI also supports running prompts directly from the command line with `-p`, and the main agent can delegate to custom subagents when it decides that helps. ([GitHub Docs][1])

What is **not** guaranteed is a hard-coded “agent chain runner” that deterministically invokes your four custom agents in sequence unless you explicitly direct it. The CLI may handle part of the work in the main agent and may delegate to subagents when it judges that useful. ([GitHub Docs][2])

So the best practical answer is:

## Best one-command approach

Run a single prompt that explicitly tells Copilot the sequence you want.

### Option A — single interactive/autopilot session

Start an interactive session and let it run with broad permissions if you’re comfortable:

```bash
copilot --allow-all --max-autopilot-continues 10
```

Then give it this prompt:

```text
Use this workflow for ticket ABC-123:

Stage 1:
Use the Requirement Author agent to read the Jira ticket and any linked external documentation.
Use the ticket-context-gathering skill if relevant.
Create or update:
- docs/tickets/ABC-123/01-requirement-definition.md

Rules for Stage 1:
- Treat the Jira ticket as the initial source of truth
- Normalize the ticket into a structured requirement without changing business intent
- Improve acceptance criteria so they are testable
- Include a concise Mermaid diagram only if it materially improves understanding

Stage 2:
Use the Solution Analyzer agent.
Read docs/tickets/ABC-123/01-requirement-definition.md and inspect the repository.
Create or update:
- docs/tickets/ABC-123/02-functional-analysis.md
- docs/tickets/ABC-123/03-technical-analysis.md
- docs/tickets/ABC-123/04-risk-analysis.md
- docs/tickets/ABC-123/05-implementation-plan.md
- docs/tickets/ABC-123/06-implementation-slices.md

Rules for Stage 2:
- Use implementation-slicing
- Prefer small, reviewable slices
- Mention Bruno updates in slices if endpoints are affected
- Do not implement code yet

At the end:
- Summarize what was created
- List open questions
- Show the proposed slice order
- Stop after Stage 2
```

Why this works:

* autopilot is designed for multi-step tasks once you give the initial instruction. ([GitHub Docs][1])
* custom agents can be invoked explicitly by name or selected directly, and the CLI can also infer/delegate when appropriate. ([GitHub Docs][2])

## Option B — literal one-shell-command with `-p`

If you want it as a single shell command:

```bash
copilot -p "Use this workflow for ticket ABC-123:

Stage 1:
Use the Requirement Author agent to read the Jira ticket and any linked external documentation.
Use the ticket-context-gathering skill if relevant.
Create or update:
- docs/tickets/ABC-123/01-requirement-definition.md

Rules for Stage 1:
- Treat the Jira ticket as the initial source of truth
- Normalize the ticket into a structured requirement without changing business intent
- Improve acceptance criteria so they are testable
- Include a concise Mermaid diagram only if it materially improves understanding

Stage 2:
Use the Solution Analyzer agent.
Read docs/tickets/ABC-123/01-requirement-definition.md and inspect the repository.
Create or update:
- docs/tickets/ABC-123/02-functional-analysis.md
- docs/tickets/ABC-123/03-technical-analysis.md
- docs/tickets/ABC-123/04-risk-analysis.md
- docs/tickets/ABC-123/05-implementation-plan.md
- docs/tickets/ABC-123/06-implementation-slices.md

Rules for Stage 2:
- Use implementation-slicing
- Prefer small, reviewable slices
- Mention Bruno updates in slices if endpoints are affected
- Do not implement code yet

At the end:
- Summarize what was created
- List open questions
- Show the proposed slice order
- Stop after Stage 2"
```

`-p` is the documented way to run a Copilot CLI prompt directly from the command line or from a script. ([GitHub Docs][3])

## The version I recommend most

For reliability, I would use **interactive mode + autopilot** rather than a blind one-liner, because GitHub explicitly notes autopilot is best for **clear, well-defined tasks**, and your two-stage prompt is well-defined enough. It is also how GitHub suggests handling larger multi-step work after planning. ([GitHub Docs][1])

## Even better: a reusable shell script

If you want this often, wrap it:

```bash
#!/usr/bin/env bash
set -euo pipefail

TICKET="${1:?Usage: ./plan-ticket.sh ABC-123}"

copilot -p "Use this workflow for ticket ${TICKET}:

Stage 1:
Use the Requirement Author agent to read the Jira ticket and any linked external documentation.
Use the ticket-context-gathering skill if relevant.
Create or update:
- docs/tickets/${TICKET}/01-requirement-definition.md

Rules for Stage 1:
- Treat the Jira ticket as the initial source of truth
- Normalize the ticket into a structured requirement without changing business intent
- Improve acceptance criteria so they are testable
- Include a concise Mermaid diagram only if it materially improves understanding

Stage 2:
Use the Solution Analyzer agent.
Read docs/tickets/${TICKET}/01-requirement-definition.md and inspect the repository.
Create or update:
- docs/tickets/${TICKET}/02-functional-analysis.md
- docs/tickets/${TICKET}/03-technical-analysis.md
- docs/tickets/${TICKET}/04-risk-analysis.md
- docs/tickets/${TICKET}/05-implementation-plan.md
- docs/tickets/${TICKET}/06-implementation-slices.md

Rules for Stage 2:
- Use implementation-slicing
- Prefer small, reviewable slices
- Mention Bruno updates in slices if endpoints are affected
- Do not implement code yet

At the end:
- Summarize what was created
- List open questions
- Show the proposed slice order
- Stop after Stage 2"
```

## My honest recommendation

Use the “one-command flow” for **Stage 1 + Stage 2 only**. That’s the sweet spot.

I would **not** extend the same one-shot prompt through implementation and validation, because autopilot is less ideal for open-ended feature development without tight supervision, and GitHub’s guidance says autopilot is best for well-defined tasks rather than ambiguous exploratory work. ([GitHub Docs][1])

The strongest operating model is:

* one command for **Jira → Requirement → Analysis → Slices**
* then one command per **implementation slice**
* then one command for **validation**

If you want, next I can give you the matching **one-command prompt for Stage 3**, so you can run: `implement-slice.sh ABC-123 02`.

[1]: https://docs.github.com/en/copilot/concepts/agents/copilot-cli/autopilot "Allowing GitHub Copilot CLI to work autonomously - GitHub Docs"
[2]: https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli-agents/invoke-custom-agents "Invoking custom agents - GitHub Docs"
[3]: https://docs.github.com/en/copilot/how-tos/copilot-cli/automate-copilot-cli/quickstart "Quickstart for automating with GitHub Copilot CLI - GitHub Docs"


-----------------

Here is the exact `.github/skills/` structure and the `SKILL.md` files, aligned with the agents and instructions you already created.

## Folder structure

```text
.github/
  skills/
    ticket-context-gathering/
      SKILL.md
    mermaid-requirement-diagrams/
      SKILL.md
    implementation-slicing/
      SKILL.md
    spring-endpoint-with-bruno/
      SKILL.md
    validation-evidence/
      SKILL.md
```

---

## `.github/skills/ticket-context-gathering/SKILL.md`

```md
---
name: Ticket Context Gathering
description: Gather and normalize ticket and documentation context before requirement writing, analysis, or validation.
---

# Ticket Context Gathering

## Purpose

Use this skill to gather the real context behind a task before writing requirements, planning implementation, or validating delivered work.

This skill is especially useful when the request references:
- a Jira ticket
- a Confluence or wiki page
- external notes or linked documentation
- prior discussion outside the repository
- acceptance criteria that may exist elsewhere

This skill helps transform fragmented context into a clear working summary.

---

## When to use

Use this skill when:
- the user references a ticket id, issue, story, epic, or bug
- the user references Confluence, wiki pages, design docs, or business notes
- the request says things like:
  - analyze this ticket
  - structure this requirement
  - plan this feature
  - validate this implementation against the original ticket
- important context is likely outside the repository
- acceptance criteria are incomplete or scattered

Do not use this skill when:
- the full requirement is already clearly present in repository files
- the task is a small isolated code fix with no relevant external context
- the prompt explicitly says to ignore external systems

---

## Inputs

Typical inputs:
- Jira ticket key, URL, summary, or pasted content
- Confluence/wiki link or pasted documentation
- raw business input
- acceptance criteria text
- notes from chat, email, or meetings
- links to related tickets or documents

---

## Expected outputs

This skill should produce a normalized context summary that can be used by other stages.

Typical outputs include:
- concise problem summary
- business goal
- known scope
- known out-of-scope items
- acceptance criteria found
- linked documentation summary
- assumptions inferred
- open questions
- dependencies or external constraints

This output usually feeds:
- `docs/tickets/<ticket>/01-requirement-definition.md`
- `docs/tickets/<ticket>/02-functional-analysis.md`
- `docs/tickets/<ticket>/03-technical-analysis.md`

---

## MCP usage guidance

If MCP-backed tools are available, use them when relevant.

Preferred usage:
- use Jira MCP to read the ticket title, description, acceptance criteria, comments, linked issues, and status
- use Confluence MCP to read referenced pages, requirements, design notes, and supporting documentation
- optionally use other knowledge tools if they are explicitly relevant

When external tools are used:
- separate retrieved facts from your own assumptions
- summarize only what is relevant to the current task
- call out contradictions between sources
- note when a linked source appears outdated or incomplete

Do not fetch external data blindly. Use only what improves the current requirement, analysis, or validation task.

---

## Working method

1. Identify the main source of truth:
   - ticket
   - linked documentation
   - user prompt
   - repository docs
2. Collect the minimum relevant context needed.
3. Normalize the information into:
   - problem
   - goal
   - scope
   - rules
   - constraints
   - acceptance criteria
   - unknowns
4. Highlight ambiguities instead of inventing facts.
5. Hand off a concise structured summary to the active agent or document.

---

## Quality rules

- Prefer concise structured summaries over long dumps of source material.
- Be explicit about what came from external sources and what is inferred.
- Preserve business meaning.
- Do not start designing the implementation unless the current stage requires it.
- Do not silently resolve contradictions; surface them clearly.
- When acceptance criteria are weak, propose improved testable wording but mark it as proposed.

---

## Recommended output shape

Use a structure like:

## Context Summary
## Source Signals
## Confirmed Acceptance Criteria
## Constraints
## Assumptions
## Open Questions
## Recommended Next Step

Keep it brief and actionable.

---

## Hand-off guidance

When used during:
- Stage 1, feed the result into `01-requirement-definition.md`
- Stage 2, feed the result into functional and technical analysis
- Stage 4, use it to recover original expectations for traceability
```

---

## `.github/skills/mermaid-requirement-diagrams/SKILL.md`

```md
---
name: Mermaid Requirement Diagrams
description: Propose and generate concise Mermaid diagrams when they improve requirement or analysis clarity.
---

# Mermaid Requirement Diagrams

## Purpose

Use this skill to decide whether a Mermaid diagram would make a requirement or analysis easier to understand, and if so, generate the most appropriate diagram.

This skill is primarily intended for:
- requirement definition
- functional analysis
- technical analysis
- validation summaries where a flow or interaction view adds clarity

The goal is not to decorate documents. The goal is to clarify behavior.

---

## When to use

Use this skill when:
- the requirement describes a multi-step business process
- there are decision branches
- multiple actors or systems interact
- there is a lifecycle or status transition
- a user flow is easier to understand visually
- the prompt explicitly asks for a Mermaid diagram

Do not use this skill when:
- the requirement is trivial and a diagram adds no value
- the diagram would repeat obvious information without clarifying anything
- the requirement is too incomplete to produce a meaningful diagram

If a diagram is not useful, say so briefly and skip it.

---

## Inputs

Typical inputs:
- raw business process description
- `01-requirement-definition.md`
- `02-functional-analysis.md`
- `03-technical-analysis.md`
- API interaction notes
- business rules
- actor/system lists

---

## Expected outputs

The skill should either:
1. propose one concise Mermaid diagram and generate it, or
2. explain briefly why no diagram is warranted

Typical output target:
- `docs/tickets/<ticket>/01-requirement-definition.md`
- optionally `02-functional-analysis.md`
- occasionally `03-technical-analysis.md`

---

## Diagram selection rules

Choose the smallest useful diagram type.

### Use `flowchart`
For:
- business process flow
- branching logic
- validation path
- happy path + alternate path

### Use `sequenceDiagram`
For:
- actor to system interactions
- API calls between components
- multi-system message flow
- request/response choreography

### Use `stateDiagram-v2`
For:
- entity lifecycle
- status changes
- approval states
- transitions and terminal states

### Use `erDiagram`
Only when:
- the requirement clearly introduces or changes meaningful domain relationships
- data structure understanding is essential to the requirement

Do not default to ER diagrams for simple field additions.

---

## Quality rules

- Prefer one good diagram over several mediocre ones.
- Keep diagrams compact and readable.
- Use business-friendly labels in requirement documents.
- Avoid over-modeling.
- Ensure the diagram matches the text and does not introduce new behavior.
- If something is inferred, mention that it is a proposed interpretation.

---

## Placement guidance

When a diagram is used in the requirement file, add a section like:

## Suggested Diagram
### Mermaid - <type and purpose>

Then include the Mermaid block.

Example labels:
- Mermaid - Business Flow
- Mermaid - Interaction Sequence
- Mermaid - Status Lifecycle

---

## Example decision logic

- approval workflow -> flowchart or state diagram
- external submission to backend plus downstream systems -> sequenceDiagram
- form submission with validation branches -> flowchart
- stateful entity like DRAFT / SUBMITTED / APPROVED / REJECTED -> stateDiagram-v2

---

## Hand-off guidance

- Stage 1: use to strengthen requirement comprehension
- Stage 2: use when functional or technical analysis benefits from visual flow
- Stage 4: use sparingly, only if a validation flow needs clarification
```

---

## `.github/skills/implementation-slicing/SKILL.md`

```md
---
name: Implementation Slicing
description: Split a planned change into small, safe, reviewable implementation slices with clear scope and test expectations.
---

# Implementation Slicing

## Purpose

Use this skill to break a requirement and technical plan into small implementation slices that can be delivered one at a time with low risk.

This skill is central to incremental delivery.

A good slice:
- is easy to review
- is narrow in scope
- is independently understandable
- includes its own unit test work
- avoids mixing unrelated concerns

---

## When to use

Use this skill when:
- planning a non-trivial ticket
- producing `06-implementation-slices.md`
- the requirement affects multiple layers or modules
- the user wants step-by-step implementation
- the change should be delivered across several commits

Do not use this skill when:
- the task is truly tiny and can safely be done in one focused change
- the requirement is still too unclear and Stage 1 is not complete enough

---

## Inputs

Typical inputs:
- `docs/tickets/<ticket>/01-requirement-definition.md`
- `docs/tickets/<ticket>/03-technical-analysis.md`
- repository structure
- existing code patterns
- testing conventions
- API or persistence constraints

---

## Expected outputs

Primary output:
- `docs/tickets/<ticket>/06-implementation-slices.md`

Supporting outputs:
- slice order
- per-slice goals
- impacted repo areas
- tests to add or update
- risks/notes
- done-when criteria

---

## Slicing rules

### General rules

- Prefer several small slices over one large slice.
- Each slice should have a single dominant purpose.
- Do not combine unrelated behaviors.
- Keep future slices out of the current implementation step.
- If a slice cannot be explained simply, it is probably too large.

### Test rule

Each slice must include:
- runtime code changes
- matching unit test changes

Do not create “tests later” as a separate slice unless there is a very strong reason.

### API rule

If a slice creates or changes an endpoint, that slice should mention:
- API contract/docs updates if relevant
- Bruno request creation or update if the repo uses Bruno
- expected status code or validation behavior changes

### Dependency rule

If a slice depends on a prior slice, say so explicitly.
If possible, reduce the dependency chain.

---

## Preferred slice patterns

Prefer vertical slices when practical, for example:
- DTO + controller path + service logic + tests for one small behavior
- validation rule + error handling + tests
- repository query + service integration + tests

Use layer-based slicing only when it is clearly safer, such as:
- a preparatory schema/model update
- a shared internal refactor required before behavior changes

---

## What each slice should contain

Use this structure:

## Slice NN - <short title>

### Goal
What the slice achieves.

### Repository areas to change
Packages, modules, or files likely to be touched.

### Runtime changes
What behavior changes in code.

### Tests to add or update
Unit tests required for this slice.

### Risks / notes
Assumptions, constraints, sequencing notes.

### Done when
Concrete completion criteria.

---

## Quality rules

- Keep slices reviewable.
- Keep slice names specific.
- Be honest about uncertainty.
- Prefer extending existing patterns instead of creating new architecture.
- Align slices with likely commit boundaries.
- Avoid a “catch-all final slice” that hides too much work.

---

## Hand-off guidance

- Stage 2: use this skill to generate `06-implementation-slices.md`
- Stage 3: use it to keep implementation bounded to the current slice
- Validation: use slice boundaries to structure evidence and review
```

---

## `.github/skills/spring-endpoint-with-bruno/SKILL.md`

```md
---
name: Spring Endpoint With Bruno
description: Standardize endpoint creation or modification in Spring Boot projects, including Bruno request updates.
---

# Spring Endpoint With Bruno

## Purpose

Use this skill whenever a Spring Boot API endpoint is created or changed.

This skill ensures that endpoint work remains complete and consistent across:
- controller behavior
- request and response contracts
- service wiring
- validation and error handling
- API documentation where used
- Bruno request artifacts for manual or team verification

This skill is not for broad feature planning. It is for endpoint-level execution consistency.

---

## When to use

Use this skill when:
- creating a new REST endpoint
- changing endpoint request fields or response shape
- changing validation or HTTP status behavior
- changing controller-to-service interaction
- updating endpoint documentation
- adding a new API request to Bruno
- modifying an existing Bruno request because endpoint behavior changed

Do not use this skill when:
- there is no endpoint involved
- the change is purely internal and not exposed through an API
- the repository does not use Bruno and the task does not need request artifacts

---

## Inputs

Typical inputs:
- requirement sections describing endpoint behavior
- current implementation slice
- existing controller/service/DTO patterns
- current OpenAPI/Swagger conventions if present
- Bruno collection structure if present
- expected request examples and error cases

---

## Expected outputs

Depending on the requested change, the skill should help produce or update:
- controller endpoint method
- request/response DTOs
- service method or orchestration
- validation behavior
- error handling alignment
- API documentation annotations or contract updates
- Bruno request files
- environment variable notes if needed

This skill usually supports Stage 3 and contributes evidence to Stage 4.

---

## Working rules

### Endpoint rules

When adding or changing an endpoint:
- align the controller signature with the requirement
- keep parameter naming and payload design consistent with existing project conventions
- preserve backward compatibility unless the requirement explicitly changes behavior
- keep status codes intentional and documented

### Service rules

- route controller logic into service logic cleanly
- avoid placing business logic directly in the controller unless that is already the repo pattern and the logic is trivial
- reuse existing patterns for mapping, validation, and response handling

### Validation and error handling rules

- account for invalid input
- ensure expected errors are handled consistently with existing project patterns
- align with existing `@ControllerAdvice` or error response style if present

### API documentation rules

If the project uses OpenAPI/Swagger:
- update endpoint summary/description if needed
- keep parameter and payload documentation aligned
- reflect expected responses where appropriate

### Bruno rules

If the repository uses Bruno:
- create or update the corresponding Bruno request
- keep naming consistent with the collection structure
- include required path params, query params, headers, and body
- add or update environment references when needed
- provide at least one happy-path request
- include a negative-path request when relevant

---

## Quality rules

- Keep endpoint work minimal and focused.
- Do not change unrelated endpoints.
- Do not silently change request or response contracts.
- Keep Bruno artifacts aligned with actual endpoint behavior.
- Avoid duplicating DTOs or patterns when an existing one should be reused.
- Be explicit when a status code or validation rule changes.

---

## Hand-off guidance

- Stage 3: use this skill during implementation of any endpoint slice
- Stage 4: use the Bruno and API details produced here as validation evidence
```

---

## `.github/skills/validation-evidence/SKILL.md`

```md
---
name: Validation Evidence
description: Produce evidence-based validation and traceability for implemented work, including build, test, API, and optional external-system evidence.
---

# Validation Evidence

## Purpose

Use this skill to validate implemented work against the original requirement, the technical plan, and the observed implementation evidence.

The goal is to make validation explicit, traceable, and auditable.

This skill is especially important at the end of implementation and before a PR is considered ready.

---

## When to use

Use this skill when:
- one or more slices have been implemented
- the user asks whether the feature is ready
- a validation report must be written
- a PR needs evidence of completeness
- acceptance criteria need to be mapped to proof
- an endpoint or externally visible behavior changed

Do not use this skill when:
- Stage 1 or Stage 2 is still too incomplete to define expected behavior
- no implementation exists yet

---

## Inputs

Typical inputs:
- `docs/tickets/<ticket>/01-requirement-definition.md`
- `docs/tickets/<ticket>/05-implementation-plan.md`
- `docs/tickets/<ticket>/06-implementation-slices.md`
- changed code
- unit or integration tests
- build/test command results
- curl examples
- Bruno requests
- optional Jenkins build results
- optional logs or incident evidence

---

## Expected outputs

Primary output:
- `docs/tickets/<ticket>/07-validation-report.md`

Expected report content:
- scope validated
- acceptance criteria traceability
- planned vs implemented notes
- technical checks performed
- API / integration validation
- negative cases checked
- risks or gaps remaining
- final status:
  - PASS
  - PASS WITH NOTES
  - FAIL

---

## Core validation rules

### Evidence first

Validation must be based on evidence, not confidence language.

Separate clearly:
- verified by execution
- verified by code inspection
- inferred but not executed
- still unverified

### Acceptance criteria traceability

For each acceptance criterion:
- restate it clearly
- identify evidence
- assign status

Typical evidence:
- unit test
- integration test
- command output
- curl execution
- Bruno request
- code path reviewed

### Plan conformance

Check whether the implementation follows the intended plan and slice boundaries.
If it deviates:
- document the deviation
- assess whether it is acceptable
- do not hide it

### Negative-path validation

Where relevant, validate:
- invalid input
- not found
- conflict/duplicate conditions
- authorization failures if applicable
- error handling behavior

---

## MCP usage guidance

If MCP-backed tools are available, use them when useful.

Good examples:
- Jenkins MCP:
  - inspect recent pipeline/build status
  - summarize failed stage or successful evidence
- log or Elasticsearch MCP:
  - validate bugfix evidence
  - correlate error signatures before/after
  - confirm representative runtime behavior if this is part of the task
- Jira MCP:
  - recover or confirm original acceptance criteria if needed

Do not rely on external evidence alone when local code or tests contradict it.

---

## Recommended output shape

Use this structure:

# Validation Report

## Scope Validated
## Acceptance Criteria Traceability
## Planned vs Implemented
## Code Review Findings
## Technical Checks Performed
## API / Integration Validation
## Negative Cases Checked
## Risks or Gaps Remaining
## Final Status

---

## Quality rules

- Do not mark PASS if critical criteria are unverified.
- Be explicit about limitations.
- Prefer concise evidence statements over long narratives.
- Include exact commands or request examples when useful.
- If something could not be executed, say so and provide the exact next validation step.
- Do not implement new production code during validation unless the user explicitly asks for fixes.

---

## Hand-off guidance

- Stage 4: use this skill as the main validation method
- PR readiness: use it to summarize delivery evidence
- bugfixes: combine code/test evidence with logs or build evidence when relevant
```

---

## Small updates I recommend to what you already created

Since you asked earlier whether agents/instructions should change: yes, just lightly.

Add this section to each agent where relevant:

```md
## Skill usage

When relevant, use available repository skills to improve consistency and reuse.

Examples:
- use `ticket-context-gathering` when ticket or documentation context exists outside the repo
- use `mermaid-requirement-diagrams` when a diagram would improve requirement or analysis clarity
- use `implementation-slicing` when planning or refining slices
- use `spring-endpoint-with-bruno` when creating or modifying API endpoints
- use `validation-evidence` when generating validation output
```

And add this section near the end of `.github/copilot-instructions.md`:

```md
## Skill usage policy

This repository may include specialized skills under `.github/skills/`.

Use skills when they improve precision, consistency, or reuse of logic.

Prefer skills for:
- external context gathering
- requirement visualization with Mermaid
- implementation slicing
- endpoint standardization including Bruno
- evidence-based validation

Skills complement agents:
- agents define the working role
- skills define how a specialized subtask should be executed

Do not force a skill when it is not relevant.
```

If you want, next I can generate the **updated versions** of the 4 agents and the instructions file with those skill references already merged in.
