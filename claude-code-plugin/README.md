# PIV Workflow - Claude Code Plugin

Plan-Implement-Validate workflow orchestrator for systematic software development.

## Version

1.0.0

## Installation

### From plugin directory
```bash
claude --plugin-dir ./piv-workflow/claude-code-plugin
```

### From marketplace (when available)
```bash
claude plugin install piv-workflow@marketplace
```

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **PIV** | `/piv-workflow:piv` | Full multi-phase PIV orchestrator with PRD → PRP → Execute → Validate loop |
| **Mini PIV** | `/piv-workflow:mini-piv` | Lightweight discovery-driven builder — no PRD needed |
| **PIV Init** | `/piv-workflow:piv-init` | Project setup — creates directories and copies templates |

## Available Agents

| Agent | Name | Description |
|-------|------|-------------|
| **Executor** | `piv-workflow:piv-executor` | Implements PRP requirements with fresh context |
| **Validator** | `piv-workflow:piv-validator` | Independently verifies implementation against PRP |
| **Debugger** | `piv-workflow:piv-debugger` | Fixes specific gaps with root cause analysis |

## When to Use

| Scenario | Use This |
|----------|----------|
| Large feature with multiple phases | `/piv-workflow:piv` (full PIV) |
| PRD already exists | `/piv-workflow:piv` (full PIV) |
| Small/medium feature, no PRD | `/piv-workflow:mini-piv` (mini PIV) |
| Quick build, explore scope first | `/piv-workflow:mini-piv` (mini PIV) |
| New project setup | `/piv-workflow:piv-init` |

## Usage Examples

### Full PIV (multi-phase)
```bash
# With PRD path
/piv-workflow:piv /path/to/PRDs/PRD-feature.md 1 4

# With project path (auto-discovers PRD)
/piv-workflow:piv /path/to/project 1 3
```

### Mini PIV (quick features)
```bash
# Interactive discovery
/piv-workflow:mini-piv

# With feature name
/piv-workflow:mini-piv "add-user-auth"

# With feature name and project path
/piv-workflow:mini-piv "token-filters" /path/to/project
```

### Project Setup
```bash
/piv-workflow:piv-init /path/to/project
```

## Architecture

```
┌─────────────────────────────────────────────┐
│           PIV ORCHESTRATOR (SKILL.md)        │
│  Lean ~15% context — manages workflow only   │
├─────────────────────────────────────────────┤
│                                              │
│  For each phase:                             │
│                                              │
│  1. Research Agent → Codebase Analysis + PRP │
│  2. piv-executor  → Implement PRP            │
│  3. piv-validator → Verify independently     │
│  4. piv-debugger  → Fix gaps (max 3x)        │
│  5. Commit on PASS                           │
│                                              │
└─────────────────────────────────────────────┘
```

## Project Structure

```
claude-code-plugin/
├── .claude-plugin/
│   └── plugin.json         # Plugin manifest
├── skills/
│   ├── piv/
│   │   ├── SKILL.md        # Full PIV orchestrator
│   │   ├── references/     # Process documentation
│   │   └── assets/         # Templates
│   ├── mini-piv/
│   │   ├── SKILL.md        # Lightweight orchestrator
│   │   ├── references/     # Process documentation
│   │   └── assets/         # Templates
│   └── piv-init/
│       └── SKILL.md        # Project setup
├── agents/
│   ├── piv-executor.md     # Executor agent
│   ├── piv-validator.md    # Validator agent
│   └── piv-debugger.md     # Debugger agent
├── scripts/
│   └── piv-ralph.sh        # Bash automation
└── README.md
```

## License

MIT
