Use the Requirement Author agent.

Context:
I want to start Stage 01 of the workflow for ticket <TICKET-ID>.

Source input:
- Jira ticket: <TICKET-ID or paste content here>
- Additional business notes: <optional>
- Related context or links: <optional>

Task:
Create or update:

docs/tickets/<ticket>/01-requirement-definition.md

Instructions:
- Treat the Jira ticket or raw input as the initial source of truth
- Normalize the requirement without changing the original business intent
- Make the requirement clear, structured, and Jira-ready
- Improve acceptance criteria so they are explicit and testable
- Clearly separate confirmed facts from assumptions
- Add open questions if anything is missing or ambiguous
- Do not design the implementation yet
- Do not write production code
- If useful, use ticket-context-gathering to collect missing context from external sources
- If it materially improves understanding, include one concise Mermaid diagram using mermaid-requirement-diagrams
- Do not include a diagram if it adds no real value

Required structure:
- Title
- Background
- Problem Statement
- Business Goal
- Scope
- Out of Scope
- Actors
- Business Rules
- Main Flow
- Alternate / Error Flows
- Data / Inputs / Outputs
- Acceptance Criteria
- Assumptions
- Open Questions
- Suggested Diagram (only if useful)
- Jira Ticket Draft

At the end:
- summarize what was created or refined
- list assumptions introduced
- list open questions that remain
- explain whether a Mermaid diagram was included and why