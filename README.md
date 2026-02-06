<p align="center">
  <h1 align="center"><b>FTW</b></h1>
  <p align="center"><i>First Try Works</i></p>
  <p align="center">The context engineering workflow that actually ships working code on the first try.</p>
</p>

<p align="center">
  <a href="https://github.com/SmokeAlot420/ftw/stargazers"><img src="https://img.shields.io/github/stars/SmokeAlot420/ftw?style=flat-square" alt="GitHub Stars"></a>
  <a href="https://github.com/SmokeAlot420/ftw/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="License"></a>
  <img src="https://img.shields.io/badge/OpenClaw-skills-purple?style=flat-square" alt="OpenClaw Skills">
  <img src="https://img.shields.io/badge/Claude_Code-plugin-orange?style=flat-square" alt="Claude Code Plugin">
</p>

<p align="center">
  <b>OpenClaw:</b> <code>clawhub install ftw</code> &nbsp;|&nbsp; <b>Claude Code:</b> <code>claude --plugin-dir ./claude-code-plugin</code>
</p>

<p align="center"><i>"Vibecoding takes 47 tries. FTW takes one."</i></p>

---

## The Problem

You tell an AI coding agent to build a feature. It writes code. You run it. It breaks. You paste the error back. It "fixes" it by rewriting half the file. Now something else breaks. Three hours later you're mass-reverting commits and questioning your life choices.

This is vibecoding. And it's how most people use AI coding tools.

The failure mode isn't the model — it's the workflow. One agent tries to plan, implement, AND validate its own work. It hallucinates success. It "fixes" things by introducing new bugs. Context rots as the conversation grows. By message 40, it's forgotten what it was even building.

**FTW fixes this.** Three specialized agents. Fresh context per agent. Independent validation that doesn't trust the executor's claims — just like real code review, except it actually runs the tests.

No prompt tweaking. No hoping it works. No "let me try that again with a better prompt."

Plan. Implement. Validate. Ship.

---

## How It Works

```
                        ┌──────────────┐
                        │   YOU / PRD  │
                        └──────┬───────┘
                               │
                    ┌──────────▼──────────┐
                    │   FTW ORCHESTRATOR  │
                    │  (~15% context)     │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
     ┌────────────┐   ┌──────────────┐  ┌────────────┐
     │  RESEARCH  │   │   EXECUTOR   │  │  VALIDATOR  │
     │            │──▶│              │─▶│             │
     │ Codebase   │   │ Implements   │  │ Independent │
     │ Analysis   │   │ PRP tasks    │  │ Verification│
     │ + PRP Gen  │   │ Fresh ctx    │  │ Fresh ctx   │
     └────────────┘   └──────────────┘  └──────┬──────┘
                                               │
                                     ┌─────────┼─────────┐
                                     │         │         │
                                   PASS    GAPS_FOUND  HUMAN
                                     │         │      NEEDED
                                     ▼         ▼         ▼
                                  Commit   ┌────────┐   Ask
                                           │DEBUGGER│   User
                                           │        │
                                           │Root    │
                                           │cause   │
                                           │fix     │
                                           └───┬────┘
                                               │
                                          Re-validate
                                          (max 3x)
```

**The key insight:** The validator is independent. It gets fresh context, reads the PRP, and checks the actual code. It doesn't see the executor's conversation. It doesn't trust claims — it verifies.

---

## FTW vs Mini FTW

| | **FTW** (Full) | **Mini FTW** |
|---|---|---|
| **Use case** | Large multi-phase projects | Quick features, single-phase |
| **Input** | PRD with defined phases | Quick discovery conversation |
| **PRP Generation** | Per phase from PRD | From 5 discovery questions |
| **Phases** | Multiple (1-N) | Single |
| **Best for** | "Build an entire auth system" | "Add a delete button" |
| **Command** | `/piv` | `/mini-piv` |

Both use the same Execute → Validate → Debug pipeline. Same quality. Different entry points.

---

## Quick Start

### OpenClaw (Recommended)

```bash
# Install from ClawHub
clawhub install ftw

# Or manual install — copy skills to your OpenClaw skills directory
cp -r openclaw-skill/piv/ ~/.openclaw/skills/piv
cp -r openclaw-skill/mini-piv/ ~/.openclaw/skills/mini-piv
```

### Claude Code Plugin

```bash
# Clone the repo
git clone https://github.com/SmokeAlot420/ftw.git

# Run Claude Code with the plugin
claude --plugin-dir ./ftw/claude-code-plugin
```

### First Run

```bash
# Full FTW — multi-phase from a PRD
/piv /path/to/PRDs/my-feature.md 1 4

# Mini FTW — quick feature, no PRD needed
/mini-piv "add-user-search"

# Project setup — creates PRDs/, PRPs/, templates
/piv-init /path/to/project
```

---

## Features

**Fresh context per agent** — Each agent spawns with a clean context window. No context rot. No "I forgot what we were building." The orchestrator stays lean at ~15% context while agents get 100% fresh.

**Independent validation** — The validator doesn't see the executor's conversation. It reads the PRP, checks the actual code, runs the tests. Just like code review, except it can't be guilt-tripped into approving.

**3-cycle debug loop with escalation** — Validator finds gaps? Debugger gets the gap list and fixes root causes, not symptoms. Still broken after 3 cycles? Escalates to you instead of silently shipping broken code.

**Structured PRPs, not vibes** — Every implementation starts from a Plan-Requirements-Protocol document with explicit tasks, success criteria, and validation steps. No ambiguity, no "figure it out."

**Language-agnostic** — Solidity, TypeScript, Python, Rust, whatever. FTW orchestrates the workflow, not the language. PRP templates work with any stack.

**Dual-target distribution** — Ships as both an OpenClaw skill pack and a Claude Code plugin. Same workflow engine, two ecosystems.

---

## Commands

| Command | Description | Example |
|---------|-------------|---------|
| `/piv` | Full multi-phase orchestrator | `/piv ./PRDs/auth-system.md 1 3` |
| `/mini-piv` | Lightweight discovery-driven builder | `/mini-piv "token-filters"` |
| `/piv-init` | Project setup (creates dirs + templates) | `/piv-init /path/to/project` |

---

## Project Structure

```
ftw/
├── claude-code-plugin/          # Claude Code plugin distribution
│   ├── .claude-plugin/
│   │   └── plugin.json          # Plugin manifest
│   ├── skills/
│   │   ├── piv/                 # Full PIV orchestrator
│   │   ├── mini-piv/            # Mini PIV orchestrator
│   │   └── piv-init/            # Project setup
│   ├── agents/
│   │   ├── piv-executor.md      # Implements PRP requirements
│   │   ├── piv-validator.md     # Independent verification
│   │   └── piv-debugger.md      # Root cause debugging
│   └── scripts/
│       └── piv-ralph.sh         # Bash automation
│
├── openclaw-skill/              # OpenClaw skill distribution
│   ├── piv/                     # Full PIV skill
│   │   ├── SKILL.md
│   │   ├── references/
│   │   ├── assets/
│   │   └── scripts/
│   └── mini-piv/                # Mini PIV skill
│       ├── SKILL.md
│       ├── references/
│       └── assets/
│
└── shared/                      # Shared source definitions
    ├── templates/
    ├── agent-defs/
    ├── process-docs/
    └── scripts/
```

---

## Who This Is For

- **Solo devs** shipping features overnight and tired of babysitting AI output
- **Teams** using AI coding agents and tired of inconsistent results across developers
- **Anyone** who's watched Claude confidently rewrite working code into broken code and then gaslight you about it
- **Builders** who want structured, repeatable AI workflows instead of prompt roulette

### Not For

- People who think prompt engineering is sufficient
- "Just tell it to try harder" advocates
- Anyone satisfied with a 1-in-47 success rate

---

## How It's Different

Most AI coding workflows are single-agent monologues. One agent plans, codes, and evaluates its own work. This is like letting a developer review their own pull request — technically possible, always a bad idea.

FTW splits the workflow into specialized agents with **separation of concerns**:

1. **Research Agent** — Analyzes the codebase and generates a structured PRP
2. **Executor Agent** — Implements the PRP with fresh context (no prior conversation baggage)
3. **Validator Agent** — Independently verifies against the PRP (doesn't trust the executor)
4. **Debugger Agent** — Fixes specific gaps with root cause analysis (not "let me rewrite everything")

Each agent starts fresh. No context rot. No accumulated hallucinations. No "I already tried that" loops.

---

## License

[MIT](LICENSE)

---

<p align="center">
  <i>Stop vibecoding. Start shipping.</i>
</p>
