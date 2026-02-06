---
name: mini-piv
description: "Lightweight PIV workflow - discovery-driven feature builder. No PRD needed. Asks 5 quick questions, generates PRP from answers, then executes the full validate/debug loop. Use for small-to-medium features, quick builds, or when you want to skip PRD ceremony."
disable-model-invocation: true
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList, Read, Write, Bash, Glob, Grep, AskUserQuestion
argument-hint: "[feature-name] [project-path]"
---

# Mini PIV Ralph - Lightweight Feature Builder

## Arguments: $ARGUMENTS

Parse arguments:
```
FEATURE_NAME = $ARGUMENTS[0] or null (will ask user during discovery)
PROJECT_PATH = $ARGUMENTS[1] or current working directory
```

**Examples:**
- `/mini-piv` → Asks for feature name during discovery, uses cwd
- `/mini-piv "add-user-auth"` → Feature name provided, uses cwd
- `/mini-piv "token-filters" /path/to/project` → Both provided

---

## Philosophy: Quick & Quality

> "When you just want to build something without writing a PRD first."

Mini PIV Ralph is the lightweight sibling of the full PIV Ralph orchestrator. Same quality pipeline (Execute → Validate → Debug), but starts from a quick conversation instead of a PRD.

**You are the orchestrator** - stay lean at ~15% context, spawn fresh sub-agents for heavy lifting.

---

## Required Reading by Role

| Role | Instructions |
|------|-------------|
| Orchestrator | This file only (you're reading it now) |
| Research Agent | references/codebase-analysis.md + references/generate-prp.md |
| Executor | Use the piv-executor agent + references/execute-prp.md |
| Validator | Use the piv-validator agent |
| Debugger | Use the piv-debugger agent |

**DO NOT wing it. Sub-agents MUST read their instruction files before acting.**

---

## Visual Workflow

```
┌──────────────────────────────────────────────────────────┐
│              MINI PIV RALPH ORCHESTRATOR                    │
├──────────────────────────────────────────────────────────┤
│ 1. DISCOVERY (Orchestrator)                                │
│    Ask user 3-5 questions conversationally                 │
│    Extract: scope, touchpoints, deps, success, out-of-scope│
│                                                            │
│ 2. RESEARCH & PRP GENERATION (Fresh sub-agent)             │
│    ├─ Step 1: Codebase analysis (deep research)            │
│    ├─ Step 2: Generate PRP from discovery answers          │
│    └─ Output: PRPs/mini-{feature-name}.md                  │
│                                                            │
│ 3. EXECUTE (Executor sub-agent)                            │
│    → EXECUTION SUMMARY                                     │
│                                                            │
│ 4. VALIDATE (Validator sub-agent)                          │
│    → PASS / GAPS_FOUND / HUMAN_NEEDED                      │
│                                                            │
│ 5. DEBUG LOOP (if needed, max 3 iterations)                │
│    Spawn DEBUGGER → Re-validate → repeat or escalate       │
│                                                            │
│ 6. COMMIT (Orchestrator)                                   │
│    feat(mini): {description}                               │
└──────────────────────────────────────────────────────────┘
```

---

## Step 1: Discovery Phase

### 1a. Determine Feature Name

If `FEATURE_NAME` is not provided:
1. Check recent conversation for context
2. If clear, propose a name and confirm
3. If unclear, ask directly

Normalize to kebab-case (e.g., "User Authentication" → "user-authentication").

### 1b. Check for Existing PRP

```bash
ls -la PROJECT_PATH/PRPs/ 2>/dev/null | grep -i "mini-{FEATURE_NAME}"
```

If exists, ask: "A PRP for '{FEATURE_NAME}' already exists. Overwrite, rename, or skip to execution?"

### 1c. Ask Discovery Questions (Conversational)

Present all questions in a single message. Adapt based on context.

**Default question set:**

```
I've got a few quick questions so I can build this right:

1. **What does this feature do?** Quick rundown — what's it supposed to accomplish?

2. **Where in the codebase does it live?** Specific files, folders, components?
   (Say "not sure" if you want me to figure it out)

3. **Any specific libraries, patterns, or existing code to follow?**

4. **What does "done" look like?** 1-3 concrete success criteria.

5. **Anything explicitly OUT of scope?** What should I NOT build right now?
```

**Adapt by feature type:**
- **UI Components** — also ask about styling, props, responsive behavior
- **API Endpoints** — also ask about request/response format, auth
- **Smart Contracts** — also ask about functions, events, security
- **Integrations** — also ask about external service details, error handling

**Wait for user response**, then extract structured answers.

### 1d. Structure Discovery Answers

```yaml
feature:
  name: {FEATURE_NAME}
  scope: {Answer to Q1}
  touchpoints: {Answer to Q2}
  dependencies: {Answer to Q3}
  success_criteria: {Answer to Q4}
  out_of_scope: {Answer to Q5}
```

---

## Step 2: Research & PRP Generation

**CRITICAL: Spawn a FRESH sub-agent. Do NOT do this yourself.**

Spawn a `general-purpose` sub-agent with this prompt:

```
MINI PIV RALPH: RESEARCH & PRP GENERATION
==========================================

You are generating a PRP for a lightweight, single-phase feature.

Project root: {PROJECT_PATH}
Feature name: {FEATURE_NAME}

## Discovery Input (from user)
{paste structured YAML from Step 1d}

## Step 1: Codebase Analysis
Run deep codebase analysis for this feature.
Save to: {PROJECT_PATH}/PRPs/planning/mini-{FEATURE_NAME}-analysis.md

## Step 2: Generate PRP (analysis context still loaded)
You already have the codebase analysis — use it directly.

### Discovery → PRP Translation Guide
| Discovery Answer | PRP Section |
|-----------------|-------------|
| Feature Scope (Q1) | Goal + What |
| Touch Points (Q2) | Implementation Blueprint task locations |
| Dependencies (Q3) | Context YAML, patterns, Known Gotchas |
| Success Criteria (Q4) | Success Criteria checklist + Final Validation |
| Out of Scope (Q5) | Explicit exclusions in What section |

### Template & Output
Use the PRP template: PRPs/templates/prp_base.md
Output PRP to: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md

## Critical Rules
- Do BOTH steps yourself in sequence
- Follow the full PRP generation process
- DO NOT spawn sub-agents
```

**Wait for completion.**

---

## Step 3: Spawn EXECUTOR Sub-Agent

Use the Task tool with `subagent_type: "piv-executor"`:

```
EXECUTOR MISSION - Mini PIV Ralph
==================================

Execute the PRP at: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md
Project root: {PROJECT_PATH}

Read references/execute-prp.md for the execution process.
Follow: Load PRP → ULTRATHINK → Execute → Validate → Verify
Output EXECUTION SUMMARY with Status, Files, Tests, Issues.
```

**Wait for completion.**

---

## Step 4: Spawn VALIDATOR Sub-Agent

Use the Task tool with `subagent_type: "piv-validator"`:

```
VALIDATOR MISSION - Mini PIV Ralph
===================================

Verify implementation of: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md
Project root: {PROJECT_PATH}
Executor Summary: {paste EXECUTION SUMMARY}

Verify ALL requirements independently. Don't trust executor claims.
Output VERIFICATION REPORT with Grade, Checks, Gaps.
```

**Process result:**
- `PASS` → Step 6 (commit)
- `GAPS_FOUND` → Step 5 (debugger)
- `HUMAN_NEEDED` → Ask user

---

## Step 5: Debug Loop (Max 3 iterations)

Spawn debugger using `subagent_type: "piv-debugger"`:

```
DEBUGGER MISSION - Mini PIV - Iteration {I}
============================================

Project: {PROJECT_PATH}
PRP Path: {PROJECT_PATH}/PRPs/mini-{FEATURE_NAME}.md
Gaps: {GAPS}
Errors: {ERRORS}

Fix root causes, not symptoms. Run tests after each fix.
Output FIX REPORT with Status, Fixes Applied, Test Results.
```

After debugger: re-validate → PASS (commit) or loop (max 3) or escalate.

---

## Step 6: Smart Commit

```bash
cd PROJECT_PATH && git status && git diff --stat
```

```bash
git add -A
git commit -m "feat(mini): implement {FEATURE_NAME}

- {bullet 1}
- {bullet 2}

Built via Mini PIV Ralph

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

## Completion Summary

```
## MINI PIV RALPH COMPLETE

Feature: {FEATURE_NAME}
Project: {PROJECT_PATH}

### Artifacts
- PRP: PRPs/mini-{FEATURE_NAME}.md
- Analysis: PRPs/planning/mini-{FEATURE_NAME}-analysis.md

### Implementation
- Validation cycles: {N}
- Debug iterations: {M}

### Files Changed
{list}

### Commit
{hash}: {message}

All requirements verified and passing.
```

---

## Error Handling

### Executor Returns BLOCKED
Ask user: "Executor blocked on '{FEATURE_NAME}'. Issue: {description}. How should we proceed?"

### Validator Returns HUMAN_NEEDED
Ask user: "Validator needs guidance. Question: {details}. Please advise."

### 3 Debug Cycles Exhausted
Ask user: "'{FEATURE_NAME}' failed validation after 3 fix attempts. Persistent issues: {list}. Need your guidance."

---

## Quick Reference

| Scenario | Use This |
|----------|----------|
| Small/medium feature, no PRD | **Mini PIV Ralph** |
| Quick feature, explore scope first | **Mini PIV Ralph** |
| Large feature with phases | Full PIV Ralph (/piv) |
| PRD already exists | Full PIV Ralph (/piv) |

### File Naming
```
PRPs/mini-{feature-name}.md                  # PRP file
PRPs/planning/mini-{feature-name}-analysis.md # Codebase analysis
```
