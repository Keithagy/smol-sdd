# SmolSDD Setup Guide

## Overview

SmolSDD is a lightweight spec-driven development workflow for Claude Code. It provides:

- **Slash commands** for the full SDD workflow (spec → research → design → plan → implement)
- **Sub-agents** for parallel codebase research
- **Beads integration** for persistent context across sessions
- **Injectable project patterns** for team-specific tribal knowledge

## Installation Options

### Option 1: Copy to Project's .claude Directory

Copy the entire smol-sdd contents into your project:

```bash
# From your project root
mkdir -p .claude
cp -r /path/to/smol-sdd/* .claude/
```

Your project structure:

```
your-project/
├── .claude/
│   ├── .claude-plugin/
│   │   └── plugin.json         # Plugin manifest
│   ├── commands/               # Slash commands
│   │   ├── spec.md
│   │   ├── research.md
│   │   ├── techdesign.md
│   │   ├── plan.md
│   │   ├── taskify.md
│   │   ├── implement.md
│   │   ├── handoff.md
│   │   └── takeon.md
│   ├── agents/                 # Sub-agent definitions
│   │   ├── codebase-locator.md
│   │   ├── codebase-analyzer.md
│   │   ├── codebase-pattern-finder.md
│   │   ├── thoughts-locator.md
│   │   ├── thoughts-analyzer.md
│   │   └── web-search-researcher.md
│   ├── skills/
│   │   ├── beads-workflow/     # Core workflow encoding
│   │   │   ├── SKILL.md
│   │   │   ├── references/     # Detailed documentation
│   │   │   └── examples/       # Usage examples
│   │   └── project-patterns/   # <-- CUSTOMIZE THIS
│   │       ├── SKILL.md
│   │       ├── patterns/
│   │       └── examples/
│   └── templates/              # Document templates
│       ├── spec.md             # Specification template
│       └── bead-description.md # Bead description template
├── thoughts/                   # Created as you work
│   ├── specs/
│   ├── research/
│   ├── design/
│   ├── plans/
│   └── handoffs/
└── .beads/                     # Created by bd init
```

### Option 2: Use as Claude Code Plugin (Advanced)

If you want to use smol-sdd across multiple projects without copying:

1. Add to your global Claude Code plugins configuration
2. Override `project-patterns` skill per-project in `.claude/skills/`

## Post-Installation Setup

### 1. Initialize Beads

SmolSDD uses [beads](https://github.com/steveyegge/beads) for persistent context:

```bash
# Install beads CLI if not already installed
# Then initialize in your project
bd init
```

### 2. Create thoughts Directory Structure

```bash
mkdir -p thoughts/{specs,research,design,plans,handoffs}
```

Add a README to document your structure:

```bash
cat > thoughts/README.md << 'EOF'
# Thoughts Directory

Project documentation and artifacts for the SDD workflow.

## Structure

- `specs/` - Feature specifications
- `research/` - Codebase research documents
- `design/` - Technical design documents
- `plans/` - Implementation plans
- `handoffs/` - Session handoff beads

## Naming Convention

`YYYY-MM-DD-<bead-id>-<description>.md`

Example: `2025-01-15-bead-abc-auth-validation.md`
EOF
```

### 3. Customize Project Patterns (IMPORTANT)

The `project-patterns` skill is a **template** for your team's tribal knowledge. You MUST customize it:

1. Edit `skills/project-patterns/SKILL.md`:
   - Update the decision trees for your architecture
   - Add your service topology
   - Document your core libraries

2. Edit `skills/project-patterns/patterns/`:
   - `state-management.md` - Where state lives in YOUR system
   - `testing-patterns.md` - YOUR test commands and conventions
   - Add patterns specific to your tech stack

3. Edit `skills/project-patterns/examples/pitfalls.md`:
   - Document mistakes that have caused issues in YOUR codebase
   - Include real examples from post-mortems

**If you don't customize project-patterns, agents won't have context about your specific codebase conventions.**

## Available Commands

| Command       | Description                              |
| ------------- | ---------------------------------------- |
| `/spec`       | Create or iterate on a specification     |
| `/research`   | Research codebase to answer questions    |
| `/techdesign` | Create or iterate on technical design    |
| `/plan`       | Create or iterate on implementation plan |
| `/taskify`    | Break plan into beads task graph         |
| `/implement`  | Complete or verify implementation tasks  |
| `/handoff`    | Create handoff for multi-session work    |
| `/takeon`     | Resume work from a handoff               |

## Available Agents

These agents are spawned by commands for parallel research:

| Agent                     | Purpose                          |
| ------------------------- | -------------------------------- |
| `codebase-locator`        | Find WHERE files live            |
| `codebase-analyzer`       | Understand HOW code works        |
| `codebase-pattern-finder` | Find existing patterns to follow |
| `thoughts-locator`        | Find documents in thoughts/      |
| `thoughts-analyzer`       | Extract insights from documents  |
| `web-search-researcher`   | Research external resources      |

## Workflow Overview

### Full Ceremony (Complex Features)

```
1. /spec           → Create specification (epic bead)
2. /research       → Answer research questions
3. /techdesign     → Design architecture
4. /plan           → Create implementation plan
5. /taskify        → Break into task beads
6. /implement      → Execute tasks
7. /implement verify → Validate completion
```

### Lightweight Path (Simple Tasks)

```
bd create "Fix bug X" --type=task
/implement bead-xyz
bd close bead-xyz
```

## Beads Workflow States

For epics following full ceremony:

```
open → needs_spec → needs_research → researching → needs_research_review
  → needs_design → designing → needs_design_review
  → needs_plan → planning → needs_plan_review
  → needs_task_breakdown → needs_implementation → implementing
  → needs_implementation_review → testing → closed
```

Human approval gates:

- `needs_research_review` → `needs_design`
- `needs_design_review` → `needs_plan`
- `needs_plan_review` → `needs_task_breakdown`
- `needs_implementation_review` → `testing`

## Injecting Tribal Knowledge

The key to effective use is properly configuring `project-patterns`. Here's an example for a Node.js/TypeScript project:

### Example: skills/project-patterns/SKILL.md

```markdown
---
name: project-patterns
description: Patterns for our Express/TypeScript backend
---

# Project Patterns

## Quick Decision Trees

### Where Should State Live?
```

Is it user-facing data?
├─ Yes → PostgreSQL (via Prisma)
└─ No
├─ Is it session/cache? → Redis
└─ Is it temporary? → In-memory

```

### Service Topology

```

API Gateway (Express) → Services → Repositories → Database
↓
Redis Cache

```

## Core Libraries

- **Types:** `src/types/`
- **Utilities:** `src/lib/`
- **Validation:** `src/middleware/validation.ts`

## Common File Patterns

- Controllers: `src/controllers/*Controller.ts`
- Services: `src/services/*Service.ts`
- Repositories: `src/repositories/*Repository.ts`
- Tests: `src/**/*.test.ts`
```

## Session Continuity

### Creating Handoffs

When ending a session with incomplete work:

```
/handoff
```

This creates a bead with:

- Current status
- Key file references
- Learnings
- Next steps

### Resuming Work

In a new session:

```
/takeon bead-xyz
```

The agent will:

1. Claim the handoff bead
2. Read all context
3. Verify codebase state
4. Present next actions

## Commit Conventions

SmolSDD uses bead IDs in commits for traceability:

```bash
# Artifacts (use epic bead ID)
git commit -m "[bead-abc] Add spec for auth feature"

# Implementation (use task bead ID)
git commit -m "[bead-xyz] Add token validation"
```

## Troubleshooting

### Commands Not Appearing

- Verify `.claude-plugin/plugin.json` exists
- Check file extensions are `.md`
- Restart Claude Code session

### Agents Not Working

- Verify agent files are in `agents/` directory
- Check YAML frontmatter syntax
- Ensure `tools:` field lists valid tools

### Beads Not Syncing

- Run `bd doctor` to check setup
- Verify `.beads/` directory exists
- Check git status for uncommitted changes

## Philosophy

> "Institutional-wisdom / tribal-knowledge context needs to be injectable. This toolkit should contain only the _workflow_."

SmolSDD separates:

- **Generic workflow** (commands, agents, beads-workflow skill)
- **Project-specific knowledge** (project-patterns skill)

This allows teams to share workflow while maintaining their own conventions.
Ultimately, this is just a pile of markdown. Use it as a jumping-off point for your own setup!
