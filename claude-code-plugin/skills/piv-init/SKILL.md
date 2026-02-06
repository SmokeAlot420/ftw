---
name: piv-init
description: Initialize a project for PIV workflow - creates PRDs/, PRPs/, PRPs/templates/ directories and copies default PRP template.
disable-model-invocation: true
argument-hint: "[PROJECT_PATH]"
---

# PIV Init - Project Setup

## Arguments: $ARGUMENTS

Parse arguments:
```
PROJECT_PATH = $ARGUMENTS[0] or current working directory
```

## Setup Process

Initialize the current project for the PIV (Plan-Implement-Validate) workflow by creating the required directory structure and copying default templates.

### Step 1: Create Directories

```bash
mkdir -p PROJECT_PATH/PRDs
mkdir -p PROJECT_PATH/PRPs/templates
mkdir -p PROJECT_PATH/PRPs/planning
```

### Step 2: Copy PRP Template

Check if `PROJECT_PATH/PRPs/templates/prp_base.md` already exists:
- **If it exists**: Tell the user "PRP template already exists at PRPs/templates/prp_base.md - skipping to preserve your customizations."
- **If it doesn't exist**: Read the bundled template from `assets/prp_base.md` (relative to this skill) and write it to `PROJECT_PATH/PRPs/templates/prp_base.md`.

### Step 3: Create WORKFLOW.md

Check if `PROJECT_PATH/WORKFLOW.md` already exists:
- **If it exists**: Tell the user "WORKFLOW.md already exists - skipping to preserve your progress."
- **If it doesn't exist**: Read the bundled template from `assets/workflow-template.md` (relative to this skill's parent `piv/` directory) and write it to `PROJECT_PATH/WORKFLOW.md`. Replace `[PROJECT_NAME]` with the directory name and `[DATE]` with today's date.

### Step 4: Confirm

Output:
```
## PIV Init Complete

Project: PROJECT_PATH

Created:
- PRDs/              (Product Requirements Documents)
- PRPs/              (Project Requirement Plans)
- PRPs/templates/    (PRP templates)
- PRPs/planning/     (Codebase analysis outputs)
- WORKFLOW.md        (Phase tracking)

Next steps:
1. Create a PRD: /piv-workflow:piv with a PRD path, or create one manually in PRDs/
2. Run full PIV: /piv-workflow:piv [PRD_PATH]
3. Run quick PIV: /piv-workflow:mini-piv [feature-name]
```
