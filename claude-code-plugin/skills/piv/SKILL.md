---
name: piv
description: PIV workflow orchestrator - Plan, Implement, Validate loop for systematic multi-phase software development. Use when building features phase-by-phase with PRPs, running automated validation, or orchestrating multi-agent development workflows. Triggers on: PIV, PRP, PRD, plan-implement-validate, multi-phase development, orchestrate workflow.
disable-model-invocation: true
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList, Read, Write, Bash, Glob, Grep
argument-hint: "[PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]"
---

# PIV Ralph Orchestrator

## Arguments: $ARGUMENTS

Parse arguments using this logic:

### PRD Path Mode (first argument ends with `.md`)

If the first argument ends with `.md`, it's a direct path to a PRD file:
- `PRD_PATH` - Direct path to the PRD file
- `PROJECT_PATH` - Derived by going up from PRDs/ folder
- `START_PHASE` - Second argument (default: 1)
- `END_PHASE` - Third argument (default: auto-detect from PRD)

### Project Path Mode

If the first argument does NOT end with `.md`:
- `PROJECT_PATH` - Absolute path to project (default: current working directory)
- `START_PHASE` - Second argument (default: 1)
- `END_PHASE` - Third argument (default: 4)
- `PRD_PATH` - Auto-discover from `PROJECT_PATH/PRDs/` folder

### Detection Logic

```
If $ARGUMENTS[0] ends with ".md":
  PRD_PATH = $ARGUMENTS[0]
  PROJECT_PATH = dirname(dirname(PRD_PATH))
  START_PHASE = $ARGUMENTS[1] or 1
  END_PHASE = $ARGUMENTS[2] or auto-detect from PRD
  PRD_NAME = basename without extension
Else:
  PROJECT_PATH = $ARGUMENTS[0] or current working directory
  START_PHASE = $ARGUMENTS[1] or 1
  END_PHASE = $ARGUMENTS[2] or 4
  PRD_PATH = auto-discover from PROJECT_PATH/PRDs/
  PRD_NAME = discovered PRD basename
```

### Auto-Detect Phases from PRD

When PRD_PATH is specified, scan the PRD for phase sections:
1. Look for: `## Phase N:`, `### Phase N:`, `**Phase N:**`, `Phase N:`
2. Set END_PHASE to highest phase found (if not specified)

---

## Required Reading by Role

**CRITICAL: Each role MUST read their instruction files before acting.**

| Role | Instructions |
|------|-------------|
| PRD Creation (if needed) | Read references/create-prd.md |
| Orchestrator (PRP Generation) | Read references/generate-prp.md |
| Executor | Use the piv-executor agent + Read references/execute-prp.md |
| Validator | Use the piv-validator agent |
| Debugger | Use the piv-debugger agent |

**DO NOT wing it. Follow the established processes.**

**Prerequisite:** A PRD must exist before running PIV Ralph. If no PRD exists, tell the user to create one first.

---

## Orchestrator Philosophy

> "Context budget: ~15% orchestrator, 100% fresh per subagent"

You are the **orchestrator**. You stay lean and manage workflow. You DO NOT execute PRPs yourself - you spawn specialized sub-agents with fresh context for each task.

**Sub-Agent Types:**
- **Executor** (`piv-executor`) - Implements PRP requirements
- **Validator** (`piv-validator`) - Independently verifies implementation
- **Debugger** (`piv-debugger`) - Fixes specific gaps

---

## Phase Workflow

For each phase from START_PHASE to END_PHASE:

### Step 1: Check/Generate PRP

#### Step 1a: Check for existing PRP
```bash
ls -la PROJECT_PATH/PRPs/ 2>/dev/null | grep -i "phase.*N\|pN\|p-N"
```
If a PRP already exists for this phase, skip to Step 2.

#### Step 1b: Spawn Fresh Research Agent for PRP Generation

**CRITICAL: Do NOT generate the PRP yourself. Spawn a FRESH sub-agent with clean context that performs codebase analysis AND PRP generation in sequence.**

Before spawning, the orchestrator must:
1. Read the PRD at PRD_PATH
2. Find the Phase N section
3. Extract the phase scope (title, deliverables, validation criteria)
4. Pass this extracted scope to the fresh agent

Spawn a `general-purpose` sub-agent with this prompt:

```
RESEARCH & PRP GENERATION MISSION - Phase {N}
==============================================

You are generating a PRP for Phase {N}. You have fresh context — use it wisely.

Project root: {PROJECT_PATH}
PRD Path: {PRD_PATH}

## Phase {N} Scope (from PRD)
{paste phase title, deliverables, and validation criteria}

## Step 1: Codebase Analysis
Read and follow the codebase analysis process doc.
Run deep codebase analysis for: {phase feature description}
Save analysis to: {PROJECT_PATH}/PRPs/planning/{PRD_NAME}-phase-{N}-analysis.md

## Step 2: Generate PRP (analysis context still loaded)
Read and follow the PRP generation process doc.
You already have the codebase analysis in your context — use it directly.
DO NOT spawn a sub-agent for this. You do it yourself.
Output PRP to: {PROJECT_PATH}/PRPs/PRP-{PRD_NAME}-phase-{N}.md

## Critical Rules
- Do BOTH steps yourself in sequence
- Your analysis context feeds directly into PRP quality
- Follow the full generate-prp process (template, quality gates, info density)
- The PRP template is at: PRPs/templates/prp_base.md
- DO NOT spawn sub-agents for either step
```

**Wait for the research agent to complete** before proceeding.

### Step 2: Spawn EXECUTOR Sub-Agent

Use the Task tool with `subagent_type: "piv-executor"`:

```
EXECUTOR MISSION - Phase {N}
============================

PRP Path: {PRP_PATH}
Project: {PROJECT_PATH}

Read the PRP execution process doc at references/execute-prp.md, then execute the PRP.
Follow: Load PRP → ULTRATHINK → Execute → Validate → Verify
Output EXECUTION SUMMARY with Status, Files, Tests, Issues.
```

**Wait for executor to complete.**

### Step 3: Spawn VALIDATOR Sub-Agent

Use the Task tool with `subagent_type: "piv-validator"`:

```
VALIDATOR MISSION - Phase {N}
=============================

PRP Path: {PRP_PATH}
Project: {PROJECT_PATH}
Executor Summary: {SUMMARY}

Verify ALL requirements independently. Don't trust executor claims.
Output VERIFICATION REPORT with Grade, Checks, Gaps.
```

**Process validator result:**
- `PASS` → Proceed to Step 5 (commit)
- `GAPS_FOUND` → Proceed to Step 4 (debugger)
- `HUMAN_NEEDED` → Ask user for guidance

### Step 4: Debug Loop (Max 3 iterations)

If validator found gaps, spawn debugger sub-agent using `subagent_type: "piv-debugger"`:

```
DEBUGGER MISSION - Phase {N} - Iteration {I}
============================================

Project: {PROJECT_PATH}
PRP Path: {PRP_PATH}
Gaps: {GAPS}
Errors: {ERRORS}

Fix root causes, not symptoms. Run tests after each fix.
Output FIX REPORT with Status, Fixes Applied, Test Results.
```

After debugger completes:
- Re-spawn **validator** to verify fixes
- If PASS → proceed to commit
- If GAPS_FOUND again → spawn debugger (up to 3 total iterations)
- After 3 iterations → escalate to user

### Step 5: Smart Commit (Orchestrator does this)

After validation passes:
```bash
cd PROJECT_PATH
git status
git diff --stat
```

Create semantic commit:
- Format: `feat/fix/refactor(scope): description`
- Add: `Co-Authored-By: Claude <noreply@anthropic.com>`

### Step 6: Update Progress

Update `PROJECT_PATH/WORKFLOW.md`:
- Mark phase N as complete
- Note validation results
- Record any issues or observations

### Step 7: Next Phase

Increment phase counter. If more phases remain, loop back to Step 1.

---

## Error Handling

### No PRD Found
If no PRD exists in `PROJECT_PATH/PRDs/`:
- Tell user: "No PRD found. Please create one first, then re-run PIV."

### Executor Returns BLOCKED
Ask user: "Executor blocked on phase N. Issue: [description]. How should we proceed?"

### Validator Returns HUMAN_NEEDED
Ask user: "Validator needs guidance on phase N. Question: [details]. Please advise."

### 3 Debug Cycles Exhausted
Ask user: "Phase N failed validation after 3 fix attempts. Persistent issues: [list]. Need your guidance."

---

## Completion

When all phases are complete, output:
```
## PIV RALPH COMPLETE

Phases Completed: START to END
Total Commits: N
Validation Cycles: M

### Phase Summary:
- Phase 1: [feature] - validated in N cycles
- Phase 2: [feature] - validated in N cycles
...

All phases successfully implemented and validated.
```

---

## Visual Workflow

```
┌──────────────────────────────────────────────────────────┐
│              PIV RALPH ORCHESTRATOR                        │
├──────────────────────────────────────────────────────────┤
│ FOR EACH PHASE (START_PHASE to END_PHASE):                │
│   a. Check if PRP exists                                  │
│   b. If not → spawn RESEARCH AGENT (analysis + PRP gen)   │
│   c. Spawn EXECUTOR → EXECUTION SUMMARY                   │
│   d. Spawn VALIDATOR → PASS / GAPS_FOUND / HUMAN_NEEDED   │
│   e. If GAPS_FOUND → Spawn DEBUGGER (max 3x)              │
│   f. Commit on PASS                                       │
│   g. Update WORKFLOW.md                                   │
│   h. Next phase                                           │
└──────────────────────────────────────────────────────────┘
```
