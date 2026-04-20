---
name: requirement-structuring
description: Transform raw feature requests into a structured requirement specification
---

# Requirement Structuring Skill

## When to use
Use this skill when:
- you receive a raw feature request
- the requirement is unstructured
- you need to produce a formal requirement specification

## Goal
Convert raw input into a complete, structured, and testable requirement specification.

## Required Output Structure
Always produce a document with the following sections:

1. Overview
2. Current behavior
3. Desired behavior
4. Scope
   - In scope
   - Out of scope
5. Actors / consumers
6. Functional requirements
7. Business rules
8. Validation rules
9. Inputs and outputs
10. Error scenarios
11. Edge cases
12. Constraints and assumptions
13. Open questions
14. Acceptance criteria
15. Notes for design phase

## Formatting Rules
- Use numbered sections
- Use bullet points inside sections
- Use prefixes:
  - FR-x for functional requirements
  - BR-x for business rules
  - VR-x for validation rules
  - AC-x for acceptance criteria
- Keep statements explicit and testable
- Avoid vague language

## Procedure
Follow these steps:

1. Identify the business goal
2. Identify current behavior
3. Identify desired behavior
4. Define scope boundaries
5. Extract functional requirements
6. Extract business rules
7. Extract validation rules
8. Identify edge cases and error scenarios
9. List assumptions
10. List open questions
11. Generate acceptance criteria

## Handling Missing Information
If information is missing:
- DO NOT guess
- Add it under "Open questions"
- Mark unclear sections with ⚠️ Missing information

## Anti-patterns to avoid
- Jumping to implementation details
- Writing long unstructured paragraphs
- Mixing current and desired behavior
- Omitting edge cases or error scenarios