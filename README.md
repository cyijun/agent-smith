# Agent Smith Matrix

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Recursive Self-Similar Multi-Agent System —— Conflict-free Parallel Task Decomposition and Execution through Directory Isolation Protocol

**English** | [简体中文](README_CN.md)

---

## Introduction

**Agent Smith Matrix** is a general-purpose multi-agent collaboration framework that enables multiple AI agents to work in parallel without conflicts through a strict directory isolation protocol.

Core Design Principles:
- **Self-Similarity**: Each agent (Smith) follows the same protocol and can recursively spawn child agents
- **Conflict-Free Parallelism**: Directory isolation ensures multiple agents can work simultaneously without interference
- **Task Decomposition**: Automatically breaks down complex tasks into parallelizable subtasks
- **Platform Agnostic**: Can run on any system or platform that supports multi-agent execution

---

## Quick Start

### Installation

**Option 1: Claude Code Skill (Recommended)**

Copy the `smith-matrix` directory to Claude Code's skills directory:

```bash
# macOS / Linux
cp -r smith-matrix ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse smith-matrix $env:USERPROFILE\.claude\skills\
```

**Option 2: Standalone Usage**

Copy the `.smith-matrix/` directory structure to your project:

```bash
cp -r .smith-matrix-template ./my-project/.smith-matrix
```

### Usage

1. **Initialize the Matrix**

   Create the `.smith-matrix/` structure in your working directory:
   ```bash
   mkdir -p .smith-matrix/{inbox,smiths,results}
   mkdir -p .smith-matrix/smiths/smith-root/{private,outbox,children}
   ```

2. **Define the Task**

   Create a task file in the `inbox/` directory:
   ```markdown
   # task-001.md
   ## Task: AI Agent Market Research

   ### Objective
   Comprehensive understanding of AI Agent market landscape

   ### Subtasks
   1. Market trend analysis
   2. Key vendor research
   3. Technology development tracking
   4. Application scenario research
   ```

3. **Start Execution**

   The root agent reads the task and decides to execute directly or decompose and create child agents

---

## Core Concepts

### Smith (Agent)

A self-similar agent unit. Each Smith has:
- Unique ID (e.g., `smith-root`, `smith-001`)
- Level indicator (Level 0 is root, increments downward)
- Parent Smith reference (root has no parent)

### Directory Isolation Protocol

```
.smith-matrix/
├── inbox/                 # Task queue (parent writes, children read)
├── smiths/
│   ├── smith-root/        # Root agent
│   │   ├── smith.md       # Agent definition (prompt)
│   │   ├── private/       # Private workspace
│   │   ├── outbox/        # Result output
│   │   └── children/      # Child agent directory
│   └── smith-001/         # Child agent
│       ├── smith.md
│       ├── private/
│       ├── outbox/
│       └── children/
└── results/
    └── final.md           # Final result
```

**Access Control Rules**:

| Directory | Permission | Description |
|-----------|------------|-------------|
| `private/` | Self-write only | Drafts, thoughts, temporary files |
| `outbox/` | Self-write only | Final result output |
| `children/` | Self-write only | Create child agents (parent privilege) |
| `inbox/` | Parent write, child read | Task distribution queue |

### Execution Flow

```
Read task from inbox/
    ↓
Analyze task complexity
    ↓
┌─────────────┴─────────────┐
↓                           ↓
Can complete directly   Needs decomposition
    ↓                           ↓
Execute task          Design subtasks
    ↓                           ↓
Write to outbox/      Create inbox/ subtasks
    ↓                           ↓
End                   Create child agents
                              ↓
                        Wait for child results
                              ↓
                        Aggregate results
                              ↓
                        Write to outbox/
                              ↓
                        End
```

---

## Platform Integration

### Claude Code

This repository includes a complete Claude Code Skill configuration:

- **Skill Entry**: `smith-matrix/SKILL.md`
- **Trigger Phrases**: "create multi-agent system", "set up agent matrix", "decompose task for parallel execution"
- **Auto-initialization**: Automatically creates `.smith-matrix/` directory structure when triggered

### Other Platforms

Smith Matrix is an open protocol that can be implemented on:

- **AutoGen** - Using UserProxyAgent + AssistantAgent combination
- **LangGraph** - As a state machine workflow
- **CrewAI** - As Crew + Agents structure
- **Custom Systems** - Any environment supporting directory I/O and multi-processing

---

## Example Scenarios

### Market Research

Decompose complex AI Agent market research into 4 parallel subtasks:
1. Market trend analysis
2. Key vendor research
3. Technology development tracking
4. Application scenario research

→ [View Full Example](./smith-matrix/examples/market-research.md)

### Code Review

Break down large-scale code review into module-level parallel processing:
1. Data layer review
2. Business logic layer review
3. API interface layer review
4. Frontend component review

### Content Creation

Distributed collaboration for content projects:
1. Outline design
2. Section writing (multiple authors in parallel)
3. Editing and proofreading
4. Format standardization

---

## Best Practices

### Task Decomposition Principles

1. **Granularity Control**: Each subtask should be completable in 1-4 hours
2. **Independence First**: Subtasks should have low coupling and minimal dependencies
3. **Clear Interfaces**: Each task should have well-defined input and output formats
4. **Endgame Mindset**: Avoid infinite decomposition; set maximum levels (recommend no more than 3)

### Result Aggregation Tips

1. **Cross-Validation**: Check consistency between subtask results
2. **Conflict Resolution**: When conflicts are found, analyze causes and provide reasonable explanations
3. **Incremental Aggregation**: Partially aggregate results immediately after subtasks complete to avoid backlog
4. **Traceability**: Reference output paths of each subtask in aggregated results

---

## Project Structure

```
smith-matrix/
├── SKILL.md              # Claude Code Skill definition
├── smith.md              # Smith core prompt template
├── examples/             # Usage examples
│   ├── market-research.md
│   └── code-refactor.md
├── references/           # Reference materials
│   ├── concepts.md       # Core concept details
│   ├── protocol.md       # Protocol specification
│   └── best-practices.md # Best practices
└── templates/            # File templates
```

---

## Protocol Specification

### Agent Definition File (smith.md)

```yaml
---
smith_id: smith-001
parent_id: smith-root
level: 1
created_at: 2026-03-05
---

# Smith {SMITH_ID}

## Identity
- ID: {SMITH_ID}
- Parent: {PARENT_ID}
- Level: {LEVEL}

## Task
Read inbox/task-{ID}.md and execute

## Constraints
- Only write to your own private/ and outbox/
- Can create child agents under children/
- Must output to outbox/result.md upon completion
```

### Result Output Format (outbox/result.md)

```markdown
# Result: {Task Title}

## Summary
One-sentence summary of the execution result.

## Detailed Results
...

## Subtask References (if any)
- smith-xxx: Responsible for ...
- smith-yyy: Responsible for ...

## Completion Status
- [x] Completed
- Completion time: 2026-03-05 12:00:00
```

---

## Contributing

Issues and Pull Requests are welcome.

### Extension Ideas

- [ ] Visual monitoring dashboard
- [ ] Result version control
- [ ] Task priority queue
- [ ] Cross-matrix collaboration protocol

---

## License

[MIT](LICENSE) © 2026 Chen Yijun
