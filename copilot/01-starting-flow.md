# 🚀 The Best Copilot Workflow (Real-World, Not Theory)

Think in **5 phases**:

```
1. Understand → 2. Design → 3. Plan → 4. Implement → 5. Validate
```

Each phase uses **Copilot differently**.

---

# 🧠 1. UNDERSTAND (Requirement → Structured Knowledge)

### Goal

Turn messy Jira tickets into **clear, structured input**

### Tooling

* Copilot CLI (main)
* MCP (Jira + Confluence)

### What you do

👉 You NEVER jump into coding

Instead:

```bash
copilot chat
```

Prompt:

```text
You are a senior software architect.

Analyze this Jira ticket and transform it into a structured requirement.

Include:
- Functional requirements
- Non-functional considerations
- Edge cases
- Missing information
- Questions to clarify

Context:
- Spring Boot backend
- MongoDB
- REST API

Ticket:
<paste here>
```

---

### 💡 Pro tip

If you have Confluence:

```text
Use available tools to search Confluence for related documentation and include relevant constraints.
```

👉 This is where MCP shines.

---

# 🏗️ 2. DESIGN (The Most Important Step)

### Goal

Create **docs-first design**

👉 You already started doing this — this is 🔥

---

### Structure (your standard)

```
docs/requirements/XXX-feature/
```

Files:

* requerimiento.md
* analisis-funcional.md
* analisis-tecnico.md
* plan-implementacion.md
* prompt-implementacion.md
* validacion.md

---

### Prompt (you already built something very good)

Refine it slightly:

```text
You are a senior software architect and backend lead.

Generate a complete documentation set for this feature.

IMPORTANT:
- Do NOT generate code
- Make decisions, do not stay generic
- Prefer practical implementation choices

Tech stack:
- Spring Boot
- MongoDB (MongoTemplate + repositories)
- REST API with OpenAPI

Output structure:
docs/requirements/002-envio-certificado/
...
```

---

### 💡 Key mindset

👉 This phase replaces hours of thinking
👉 You review + tweak → not reinvent

---

# 🧩 3. PLAN (Turn Design into Executable Work)

### Goal

Create **step-by-step tasks Copilot can execute**

---

### Prompt

```text
Based on the approved documentation:

docs/requirements/002-envio-certificado/

Generate an implementation plan:

- Ordered steps
- Each step small and testable
- Include dependencies between steps
- Identify risks

Output as a checklist.
```

---

### Output example

```
1. Create DTOs
2. Create Entity
3. Create Repository
4. Create Service
5. Create Controller
6. Add OpenAPI annotations
7. Add Exception Handling
8. Add Unit Tests
```

---

### 💡 Why this matters

Without this → Copilot hallucinates big chunks
With this → Copilot executes like a junior dev

---

# ⚙️ 4. IMPLEMENT (Where Copilot Actually Codes)

Now you switch to **execution mode**

---

## 🧠 Golden Rule

👉 One step at a time
👉 Never: “implement everything”

---

### Example prompt

```text
Implement step 1 of the plan:

- Create DTOs for envio-certificado
- Follow project conventions
- Use Lombok
- Add validation annotations

Place everything under:
certificado/
```

---

Then:

```text
Implement step 2...
```

---

### 💡 Advanced trick

Use context:

```text
Use:
- analisis-tecnico.md
- plan-implementacion.md

as source of truth.
```

---

## 🔁 Iteration Loop

For each step:

```
Generate → Review → Fix → Commit
```

---

# 🧪 5. VALIDATE (Where Most People Fail)

### Goal

Make Copilot **prove correctness**

---

### Prompt

```text
Validate the implementation against:

- requerimiento.md
- analisis-funcional.md

Check:
- Missing scenarios
- Edge cases
- Incorrect assumptions
- API contract mismatches

List issues only.
```

---

Then:

```text
Fix all issues found.
```

---

# 🔍 BONUS: PR REVIEW FLOW (Very Powerful)

Before opening PR:

```text
Act as a senior reviewer.

Review this code considering:
- Clean code
- Spring Boot best practices
- Mongo usage
- Error handling
- Test coverage

Be strict.
```

---

# 🤖 WHERE AGENTS / SKILLS FIT

Now the key question you asked 👇

---

## 🧩 Recommended Setup

### 🧠 Agents (role-based)

1. **Architect Agent**

   * Requirement → design docs

2. **Backend Agent**

   * Spring Boot implementation

3. **Mongo Agent**

   * Aggregations, queries

4. **Test Agent**

   * Unit + integration tests

5. **Reviewer Agent**

   * PR review

---

## 🧰 Skills (reusable knowledge)

* spring-boot-standards.md
* mongo-patterns.md
* api-design-guidelines.md
* testing-guidelines.md

👉 YES — exactly what you said:

> generate from prompt → reuse across projects

---

# 🔄 FULL FLOW (Your Ideal Loop)

```
Jira Ticket
   ↓
[Copilot CLI] → Requirement Analysis
   ↓
[Copilot CLI] → Docs Generation
   ↓
[You] → Review & Adjust
   ↓
[Copilot CLI] → Implementation Plan
   ↓
Loop:
   → Implement Step
   → Review
   → Fix
   ↓
[Copilot CLI] → Validation
   ↓
[Copilot CLI] → PR Review
   ↓
Merge 🚀
```

---

# ⚠️ Common Mistakes (avoid these)

❌ Asking Copilot to “build everything”
❌ Skipping design phase
❌ Not grounding with docs
❌ Not iterating step-by-step
❌ Not validating against requirements

---

# 🔥 If you want next step

I can give you:

👉 A **real starter kit repo structure**
👉 Ready-to-use:

* agents
* skills
* prompts
* folder layout

Or…

👉 A **v2 advanced workflow** with:

* backend.instructions.md / frontend.instructions.md
* CLI automation scripts
* multi-agent orchestration

Just tell me 👍
