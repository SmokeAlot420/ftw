# Mini PIV Ralph - Lightweight Feature Builder Process

## Overview

Mini PIV Ralph is the lightweight sibling of the full PIV Ralph orchestrator. Same quality pipeline (Execute, Validate, Debug), but starts from a quick conversation instead of a PRD.

## When to Use

| Scenario | Use This |
|----------|----------|
| Small/medium feature, no PRD | **Mini PIV Ralph** |
| Quick feature, explore scope first | **Mini PIV Ralph** |
| Large feature with phases | Full PIV Ralph |
| PRD already exists | Full PIV Ralph |
| Multi-phase project | Full PIV Ralph |

## Process Flow

1. **Discovery** - Ask user 3-5 questions to understand scope
2. **Research & PRP Generation** - Codebase analysis + PRP from discovery answers
3. **Execute** - Implement PRP requirements
4. **Validate** - Independent verification
5. **Debug Loop** - Fix gaps (max 3 iterations)
6. **Commit** - Semantic commit on success

## Validation Sizing

Choose validation depth based on change scope:
- **Quick validation** (orchestrator verifies directly): <5 files changed, <100 lines, no external integrations
- **Full validation** (spawn validator sub-agent): 5+ files, 100+ lines, external APIs, security-sensitive code

## Discovery Questions

Default question set (adapt based on feature type):

1. **What does this feature do?** Quick rundown of what it accomplishes
2. **Where in the codebase does it live?** Specific files, folders, or components
3. **Any specific libraries, patterns, or existing code to follow?**
4. **What does "done" look like?** 1-3 concrete success criteria
5. **Anything explicitly OUT of scope?**

## Discovery-to-PRP Translation

| Discovery Answer | PRP Section |
|-----------------|-------------|
| Feature Scope (Q1) | Goal + What |
| Touch Points (Q2) | Implementation Blueprint task locations |
| Dependencies (Q3) | Context YAML, patterns, Known Gotchas |
| Success Criteria (Q4) | Success Criteria checklist + Final Validation |
| Out of Scope (Q5) | Explicit exclusions in What section |

## File Naming

```
PRPs/mini-{feature-name}.md                       # PRP file
PRPs/planning/mini-{feature-name}-analysis.md      # Codebase analysis
```
