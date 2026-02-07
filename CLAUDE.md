# FTW (First Try Works)

Dual-target skill distribution monorepo. Ships as both a **Claude Code plugin** and **OpenClaw skill pack**. No build system, no tests, no application code — this repo is all-markdown skill definitions and process docs.

## Architecture: Vertical Slices

```
shared/                    <- Canonical source of truth (edit here first)
├── agent-defs/            <- piv-executor.md, piv-validator.md, piv-debugger.md
├── process-docs/          <- codebase-analysis.md, generate-prp.md, execute-prp.md, create-prd.md, mini-piv-ralph.md
├── templates/             <- prp_base.md, workflow-template.md
└── scripts/               <- piv-ralph.sh

claude-code-plugin/        <- Slice 1: Claude Code distribution
├── .claude-plugin/plugin.json
├── skills/{piv,mini-piv,piv-init}/SKILL.md + references/ + assets/
├── agents/{piv-executor,piv-validator,piv-debugger}.md
└── scripts/piv-ralph.sh

openclaw-skill/            <- Slice 2: OpenClaw distribution
├── piv/SKILL.md + references/ + assets/
└── mini-piv/SKILL.md + references/ + assets/
```

Each slice is **self-contained** — it bundles everything it needs. The slices are platform-adapted copies, not symlinks.

## Shared Source Is Canonical

**Never edit distribution copies directly.** The workflow is:

1. Edit in `shared/`
2. Adapt and sync to both `claude-code-plugin/` and `openclaw-skill/`
3. Verify both slices work
4. Commit

If you find a bug in a distribution copy, fix it in `shared/` first, then propagate.

## Platform Conventions

| Concern | Claude Code (`claude-code-plugin/`) | OpenClaw (`openclaw-skill/`) |
|---------|-------------------------------------|------------------------------|
| Sub-agent spawning | `Task` tool with `subagent_type` param | `sessions_spawn` tool |
| Tool names | `Read, Write, Edit, Bash, Glob, Grep` (capitalized) | `read, write, edit, exec, grep, find` (lowercase) |
| Path resolution | Relative (framework resolves from skill dir) | `{baseDir}/` prefix (runtime variable) |
| Agent definitions | Separate `agents/` dir, YAML frontmatter (`name`, `description`, `tools`, `model: inherit`) | Inline as `references/` files, no frontmatter (plain markdown) |
| Metadata | `.claude-plugin/plugin.json` at plugin root | SKILL.md frontmatter with single-line JSON5 `metadata` field |
| Skill structure | `skills/<name>/SKILL.md` with `references/` and `assets/` subdirs | Top-level `<name>/SKILL.md` with `references/` and `assets/` subdirs |
| Skills available | `piv`, `mini-piv`, `piv-init` | `piv`, `mini-piv` |
| Sub-agent hierarchy | Flat (sub-agents cannot spawn sub-agents) | Flat (same constraint) |

## Claude Code Conventions

### plugin.json

Lives at `claude-code-plugin/.claude-plugin/plugin.json`. Only `plugin.json` goes inside `.claude-plugin/` — all other components (`skills/`, `agents/`, `scripts/`) go at the plugin root.

### Agent Frontmatter

```yaml
---
name: piv-executor
description: PIV Executor - implements PRP requirements with fresh context.
tools: Read, Write, Edit, Bash, Glob, Grep
model: inherit
---
```

Required fields: `name`, `description`, `tools`, `model: inherit`.

### Skill Frontmatter

```yaml
---
name: piv
description: PIV workflow orchestrator - ...
disable-model-invocation: true
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList, Read, Write, Bash, Glob, Grep
argument-hint: "[PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]"
---
```

### Spawning Sub-Agents

Use the `Task` tool with named `subagent_type`:
- `piv-executor` — implements PRP requirements
- `piv-validator` — independent verification
- `piv-debugger` — root cause debugging
- `general-purpose` — research/PRP generation (no custom agent needed)

## OpenClaw Conventions

### SKILL.md Frontmatter

Must include a single-line JSON5 `metadata` field:

```yaml
---
name: piv
description: "..."
user-invocable: true
disable-model-invocation: true
metadata: {"openclaw":{"emoji":"gear","homepage":"https://github.com/SmokeAlot420/ftw","requires":{"bins":["git"]},"os":["darwin","linux"]}}
---
```

### Path Resolution

All file references use `{baseDir}/` prefix — this is a runtime variable resolved by OpenClaw to the skill's root directory:

```
Read {baseDir}/references/codebase-analysis.md
```

Never use absolute paths or bare relative paths in OpenClaw skills.

### Agent Definitions

OpenClaw has no `agents/` directory. Agent definitions go in `references/` as plain markdown files (no YAML frontmatter). The SKILL.md references them inline:

```
Read {baseDir}/references/piv-executor.md for your role.
```

### Spawning Sub-Agents

Use `sessions_spawn` tool. Each spawn is non-blocking — results arrive via an announce step.

## SKILL.md Authoring Rules (Both Targets)

1. **Under 500 lines** — every SKILL.md must stay under 500 lines
2. **`disable-model-invocation: true`** — skills are user-invoked, not auto-triggered
3. **`$ARGUMENTS`** — use this variable to receive user input; parse with documented detection logic
4. **Process docs in `references/`** — heavy process instructions go in reference files, not inline
5. **Templates in `assets/`** — PRP templates, workflow templates go in assets
6. **Each skill is self-contained** — bundle all needed references and assets within the skill directory
7. **Sub-agents get fresh context** — orchestrator stays lean (~15% context), sub-agents get 100% fresh

## Skills Reference

| Skill | Command | Purpose |
|-------|---------|---------|
| `piv` | `/piv [PRD_PATH] [START] [END]` | Full multi-phase PIV orchestrator |
| `mini-piv` | `/mini-piv [feature-name] [project-path]` | Lightweight single-phase, discovery-driven |
| `piv-init` | `/piv-init [project-path]` | Project setup (creates dirs + templates) |

## Publishing

### OpenClaw

```bash
# Manual install
cp -r openclaw-skill/piv/ ~/.openclaw/skills/piv
cp -r openclaw-skill/mini-piv/ ~/.openclaw/skills/mini-piv

# ClawHub (when published)
clawhub publish
```

### Claude Code

```bash
# Run with plugin
claude --plugin-dir ./claude-code-plugin
```

## Development Workflow

1. **Edit shared/** — make changes in `shared/agent-defs/`, `shared/process-docs/`, `shared/templates/`, or `shared/scripts/`
2. **Sync to Claude Code** — adapt and copy to `claude-code-plugin/` (add YAML frontmatter to agents, use capitalized tool names, use `Task` tool for spawning, use relative paths)
3. **Sync to OpenClaw** — adapt and copy to `openclaw-skill/` (strip agent frontmatter into plain markdown references, use lowercase tool names, use `sessions_spawn` for spawning, use `{baseDir}/` paths)
4. **Verify both slices** — check that references resolve, paths are correct, tool names match platform conventions
5. **Commit** — single commit covering shared + both distribution targets

## AI Instructions

When working on this repo:

- **Read this file first** before making any changes
- **shared/ is canonical** — always start edits there
- **Platform parity** — changes must be synced to both distribution targets with correct platform adaptations
- **Respect the platform table** — tool names, paths, spawning mechanisms, and metadata formats differ between Claude Code and OpenClaw
- **No hardcoded paths** — use relative resolution (Claude Code) or `{baseDir}/` (OpenClaw)
- **Keep SKILL.md under 500 lines** — move heavy content to `references/`
- **`context: fork` has known bugs in OpenClaw** (#17283) — do not rely on it; design skills to work without it
- **Sub-agents cannot spawn sub-agents** — flat hierarchy only in both platforms
- **Test both entry points** — `/piv` (full) and `/mini-piv` (lightweight) after any workflow changes
