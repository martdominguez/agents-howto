Create a structured documentation package for this change.

Use:
- requirements-analyst
- codebase-impact-analyst
- risk-and-migration-analyst
- implementation-planner

Inputs:
- Ticket: <TICKET_KEY>

### OUTPUT STRUCTURE

Create the following files:

docs/changes/<TICKET_KEY>/

1. analysis.md
2. risks.md
3. implementation-plan.md
4. implementation.prompt.md

---

### RULES

- Do NOT implement code
- Each file must be focused and well structured
- Use clear Markdown headings
- Be concise but precise

---

### FILE DETAILS

#### analysis.md
- executive summary
- functional analysis
- technical impact
- affected artifacts

#### risks.md
- categorized risks
- mitigation strategies

#### implementation-plan.md
- ordered steps
- dependencies
- testing plan

#### implementation.prompt.md
- a ready-to-use Copilot prompt to implement the change
