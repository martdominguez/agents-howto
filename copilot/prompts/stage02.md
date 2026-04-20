Use the Solution Analyzer agent.

Context:
I want to start Stage 02 of the workflow for ticket <TICKET-ID>.

Input file:
- docs/tickets/<ticket>/01-requirement-definition.md

Task:
Read the requirement file, inspect the repository, and create or update these files:

- docs/tickets/<ticket>/02-functional-analysis.md
- docs/tickets/<ticket>/03-technical-analysis.md
- docs/tickets/<ticket>/04-risk-analysis.md
- docs/tickets/<ticket>/05-implementation-plan.md
- docs/tickets/<ticket>/06-implementation-slices.md

Instructions:
- Analyze the requirement against the real repository structure and existing code patterns
- Identify impacted modules, packages, endpoints, DTOs, persistence, validation, logging, security, integrations, and tests
- Document both functional and technical impact
- Identify risks and mitigations explicitly
- Create an implementation plan in a safe order
- Use implementation-slicing to produce small, reviewable implementation slices
- Each slice must be independently understandable and implementable
- Each slice must include the unit tests that belong to that slice
- If endpoints are affected, mention OpenAPI and Postman alignment explicitly in the relevant slices
- Optionally use ticket-context-gathering if additional external ticket or documentation context is needed
- Optionally include a concise Mermaid diagram only when it improves the analysis
- Do not implement production code yet

Important:
In 06-implementation-slices.md, for each slice include:
- Goal
- Repository areas to change
- Runtime changes
- Tests to add or update
- Risks / notes
- Done when

Also:
At the end of 05-implementation-plan.md or 06-implementation-slices.md, generate a ready-to-use implementation prompt template for Stage 03 that can be reused slice by slice.

At the end of your response:
- summarize the main impacted areas
- list the highest risks
- show the proposed slice order
- list blocking open questions
- confirm whether the Stage 03 implementation prompt template was generated