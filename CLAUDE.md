# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Smith is a **Claude Code skill** that implements a recursive self-similar multi-agent system. It enables conflict-free parallel task decomposition and execution through a strict directory isolation protocol.

This is a **documentation-only project**. There are no build scripts, test suites, package managers, or compiled artifacts. The entire "product" is a set of Markdown files that define the skill behavior, agent protocols, examples, and templates.

## Project Structure

```
agent-smith/
├── SKILL.md              # Claude Code skill entry point (trigger definitions, initialization logic)
├── smith.md              # Core agent prompt template (Chinese) — defines Smith identity, protocol, constraints
├── examples/             # Usage examples
│   ├── market-research.md
│   └── code-refactor.md
├── references/           # Reference documentation
│   ├── concepts.md       # Core concepts: Smith definition, directory isolation, recursive decomposition
│   ├── protocol.md       # Conflict-free protocol: write isolation, parent-child rules, queue consumption
│   └── best-practices.md # Task decomposition guidelines, result aggregation tips
└── templates/
    └── task.md.template  # Task file template for inbox/ tasks
```

Root-level `README.md` and `README_CN.md` are user-facing documentation, not part of the skill runtime.

## Development Workflow

### Installing the Skill for Testing

After editing any skill file, reinstall it to Claude Code's skills directory:

```bash
cp -r agent-smith ~/.claude/skills/
```

Then test by invoking one of the trigger phrases (e.g., "create multi-agent system") in a Claude Code session.

### What to Edit

- **`agent-smith/SKILL.md`** — Skill metadata, trigger conditions, initialization steps, child-agent creation logic. This is what Claude Code reads when the skill is invoked.
- **`agent-smith/smith.md`** — The agent definition template copied to each Smith's directory. Contains the core protocol, recursion limits, write constraints, and result format. Placeholders `{SMITH_ID}`, `{PARENT_ID}`, `{LEVEL}` are replaced at creation time.
- **`agent-smith/references/`** — Deep-dive docs referenced by `SKILL.md`. Keep in sync if you change protocol rules.
- **`agent-smith/examples/`** — Scenario walkthroughs. Update if you add new features or change decomposition patterns.
- **`agent-smith/templates/task.md.template`** — Task file template used when creating inbox tasks.

## Architecture

### Directory Isolation Protocol

The framework creates a `.smith-matrix/` directory tree at runtime. Each Smith has its own `inbox/`, `private/`, `outbox/`, and `children/`:

```
.smith-matrix/
├── smiths/
│   └── smith-root/        # Root agent
│       ├── smith.md       # Agent definition (read-only after creation)
│       ├── inbox/         # Task assigned by parent (or user for root)
│       ├── private/       # Agent's private workspace
│       ├── outbox/        # Final result output (result.md)
│       └── children/      # Child agent directories (parent creates)
│           └── smith-001/
│               ├── smith.md
│               ├── inbox/ # Parent writes task here
│               ├── private/
│               ├── outbox/
│               └── children/
└── results/
    └── final.md           # Root's aggregated final result
```

**Access rules**:
- Each Smith may only write to its own `private/`, `outbox/`, and `children/`.
- Each Smith may only read tasks from its own `inbox/` written by its parent.
- Parents create child directories **and** write task files to the child's `inbox/`; children never create siblings or modify parent directories.

### Recursive Self-Similarity

Every Smith is an identical agent running the same `smith.md` protocol. A Smith reads its task from its own `inbox/`, decides whether to execute directly or decompose, and if decomposing, creates child Smiths (each with their own `inbox/`) that follow the exact same rules.

### Hard Constraints (enforced by protocol)

| Constraint | Value |
|------------|-------|
| Maximum recursion depth | 3 (Level 0 root → max Level 3) |
| Maximum children per Smith | 5 |
| Level ≥ 3 behavior | Must execute directly; decomposition forbidden |

These limits are defined in `smith.md` and referenced in `SKILL.md`. Changing them requires updating both files consistently.

### Execution Flow

1. Parent creates `children/smith-{id}/` (including `inbox/`) with populated `smith.md`, then writes `children/smith-{id}/inbox/task-{id}.md`.
2. Child reads its identity from `./smith.md`, reads its task from `./inbox/`.
3. Child either executes and writes `outbox/result.md`, or decomposes further (if under limits).
4. Parent polls `children/*/outbox/result.md`, aggregates, and writes its own `outbox/result.md`.

## Language Notes

- `smith.md` and reference docs are written in **Chinese**. The skill itself (`SKILL.md`) is also primarily Chinese with some English metadata.
- `README.md` is English; `README_CN.md` is Chinese.
- When editing, maintain language consistency with the surrounding document.
