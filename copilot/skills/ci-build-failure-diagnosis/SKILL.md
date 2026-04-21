---
name: ci-build-failure-diagnosis
description: Diagnose CI/CD build failures from logs, stages, and test reports, identifying root cause, evidence, and next steps.
---

# CI Build Failure Diagnosis

## Purpose

This skill analyzes CI/CD build failures (Jenkins, GitHub Actions, GitLab CI, etc.) and determines:

- Failure type
- Root cause
- Supporting evidence
- Confidence level
- Recommended next step

It is designed to extract **signal over noise** from logs and avoid misleading cascading errors.

---

## When to use this skill

Use this skill when:

- A build has failed
- Logs, stages, or test reports are available
- The goal is to **explain why the build failed**, not just show logs

---

## Inputs expected

The agent should provide:

- Build metadata (job, branch, build number if available)
- Failed stage(s)
- Relevant log excerpts (not full logs)
- Test report summary (if applicable)

---

## Diagnosis Process

### 1. Identify the FIRST failure

- Locate the earliest failing stage or step
- Ignore downstream/cascading failures
- Focus only on the **root failure trigger**

---

### 2. Classify the failure type

Choose the most appropriate category:

- **COMPILATION_ERROR**
- **TEST_FAILURE**
- **DEPENDENCY_ERROR**
- **INFRASTRUCTURE_ERROR**
- **TIMEOUT**
- **PERMISSION_ERROR**
- **DEPLOYMENT_ERROR**
- **CONFIGURATION_ERROR**
- **UNKNOWN**

---

### 3. Extract minimal evidence

Only include **high-signal data**:

- Key error message
- Top of stack trace (3–5 lines max)
- Failing test name (if applicable)
- Relevant command or step

Avoid dumping large logs.

---

### 4. Correlate and interpret

Determine:

- Is this deterministic or flaky?
- Is it caused by:
  - code change
  - dependency change
  - environment issue
  - configuration issue

If uncertain, explicitly state ambiguity.

---

### 5. Determine confidence level

- **HIGH** → clear root cause and strong evidence
- **MEDIUM** → likely cause but partial ambiguity
- **LOW** → insufficient or conflicting evidence

---

## Heuristics (Patterns)

Use these patterns when applicable:

### Dependency issues
- "Could not resolve dependency"
- "Failed to download"
→ DEPENDENCY_ERROR

### Compilation issues
- "cannot find symbol"
- "compilation failed"
→ COMPILATION_ERROR

### Test failures
- "Tests run:"
- "AssertionError"
- "expected but was"
→ TEST_FAILURE

### Infrastructure issues
- "Connection refused"
- "Host unreachable"
- "timeout connecting"
→ INFRASTRUCTURE_ERROR

### Permission issues
- "Access denied"
- "Permission denied"
→ PERMISSION_ERROR

### Memory / resource issues
- "OutOfMemoryError"
- "Killed process"
→ INFRASTRUCTURE_ERROR

---

## Output format (MANDATORY)

Respond EXACTLY using this structure:

## Build
- Job: <job name or unknown>
- Branch: <branch or unknown>
- Build: <build number or unknown>

## Failure summary
<1–3 sentence concise explanation>

## Failure type
<ONE of the predefined types>

## Evidence
- Stage: <stage name>
- Error: <main error message>
- Snippet:
