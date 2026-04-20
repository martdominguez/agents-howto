Use the Validator agent.

Context:
I want to start Stage 04 of the workflow for ticket <TICKET-ID>.

Input files:
- docs/tickets/<ticket>/01-requirement-definition.md
- docs/tickets/<ticket>/03-technical-analysis.md
- docs/tickets/<ticket>/05-implementation-plan.md
- docs/tickets/<ticket>/06-implementation-slices.md

Task:
Validate the implemented work and create or update:

docs/tickets/<ticket>/07-validation-report.md

Instructions:
- Validate the implementation against the requirement and acceptance criteria
- Validate the implementation against the intended technical plan
- Inspect the code and the tests that were added or changed
- Use validation-evidence to map acceptance criteria to evidence
- If needed, use ticket-context-gathering only to recover original ticket context or missing acceptance criteria from external sources
- Be explicit about what was actually validated versus what was only inferred
- If endpoints were changed, include API validation evidence such as curl examples, expected request/response behavior, OpenAPI alignment, and Postman request coverage
- Review negative-path behavior where relevant
- Document remaining risks, gaps, or unverified areas
- Do not implement new production code in this stage unless I explicitly ask for fixes afterward

Required report structure:
- Scope Validated
- Acceptance Criteria Traceability
- Planned vs Implemented
- Code Review Findings
- Technical Checks Performed
- API / Integration Validation
- Negative Cases Checked
- Risks or Gaps Remaining
- Final Status

Validation rules:
- Do not mark PASS if key acceptance criteria are not verified
- Use PASS, PASS WITH NOTES, or FAIL
- If something could not be executed, say so clearly and provide the exact next validation step

At the end of your response:
- summarize final status
- list failed or partial criteria
- list recommended next actions
- state whether validation relied only on repository evidence or also on external context