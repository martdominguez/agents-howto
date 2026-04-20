---
name: mermaid-diagram-generation
description: Generate a clear and meaningful Mermaid diagram to visualize system behavior or structure
---

# Mermaid Diagram Generation Skill

## When to use
Use this skill when:
- a requirement, design, or feature needs visual clarification
- a flow, interaction, or structure would benefit from a diagram
- documenting behavior for easier understanding and validation

## Goal
Produce a **single, clear, and useful Mermaid diagram** that helps visualize the system behavior or structure.

---

## Diagram Selection Rules

Choose the most appropriate diagram type:

### 1. Flowchart (default)
Use for:
- business logic
- decision flows
- validation paths
- general process behavior

```mermaid
flowchart TD
````

---

### 2. Sequence diagram

Use for:

* API calls
* service interactions
* request/response flow between components

```mermaid
sequenceDiagram
```

---

### 3. Class diagram

Use for:

* domain modeling
* entities, DTOs, relationships

```mermaid
classDiagram
```

---

## Output Requirements

* Generate **exactly ONE diagram**
* Use valid Mermaid syntax inside a fenced code block
* Keep the diagram **simple, readable, and focused**
* Use meaningful labels (avoid generic names like A, B, C)
* Prefer top-down (`TD`) flow for readability (flowcharts)

---

## Placement

* Place the diagram under a section titled:

```md
## Diagram
```

---

## Content Rules

The diagram must:

* Represent the **desired behavior** (not only the current one)
* Show key:

  * decisions
  * flows
  * transitions
  * actors (if applicable)
* Avoid unnecessary technical details unless required
* Avoid mixing multiple concerns in one diagram

---

## Procedure

1. Identify what needs to be visualized:

   * flow
   * interaction
   * structure

2. Select the diagram type:

   * flowchart → default
   * sequence → interactions
   * class → structure

3. Extract key elements:

   * steps
   * decisions
   * actors
   * transitions

4. Build the diagram:

   * clear nodes
   * clear transitions
   * minimal complexity

5. Validate:

   * readable
   * consistent
   * aligned with the requirement

---

## Handling Missing Information

If details are unclear:

* simplify the diagram
* avoid guessing unknown flows
* reflect only confirmed or clearly inferred behavior

---

## Anti-patterns to avoid

* generating multiple diagrams
* overcomplicated diagrams with too many nodes
* mixing current and desired behavior
* using placeholder labels (A, B, Step1, etc.)
* invalid Mermaid syntax
* embedding explanations inside the diagram instead of outside