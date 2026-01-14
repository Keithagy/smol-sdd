# Bead Types: Semantic Definitions

This document defines when and how to use each bead type in the workflow.

## Core Types

### Epic

**Purpose:** Container for multi-session, multi-PR features requiring full ceremony.

**When to create:**
- Feature touches multiple services
- Requires architectural decisions
- Will span multiple sessions
- Needs spec → research → design → plan → tasks flow

**Status progression:** Full 15-state machine

**Example:**
```bash
bd create "Non-liquidatable subaccount capabilities" \
  --type epic \
  --priority 1 \
  --description "Feature to add per-subaccount capability flags"
```

**Characteristics:**
- No parent (root level)
- Contains child tasks
- Description links to artifacts (spec, research, design, plan)
- Status encodes workflow position

### Task

**Purpose:** Granular implementation unit, typically child of an epic.

**When to create:**
- Implementation unit from plan phase
- Research question from spec
- Discrete piece of work with clear success criteria

**Status progression:** `open` → `in_progress` → `closed`

**Example:**
```bash
bd create "Phase 1: Add database schema" \
  --type task \
  --parent myproject-abc \
  --priority 2
```

**Characteristics:**
- Has parent (epic or feature)
- May have dependencies (blocks/blocked-by)
- Claimable via `bd ready`
- Description follows bead template

### Feature

**Purpose:** Single-PR feature not requiring full ceremony.

**When to create:**
- Self-contained feature
- Clear implementation path
- No architectural decisions needed
- Single session work

**Status progression:** `open` → `in_progress` → `closed`

**Example:**
```bash
bd create "Add retry logic to API client" \
  --type feature \
  --priority 2
```

**Characteristics:**
- No parent (standalone)
- May spawn subtasks if complexity grows
- Lightweight path (skip spec/research/design)

### Bug

**Purpose:** Bug fix or defect correction.

**When to create:**
- Fixing incorrect behavior
- Addressing regression
- Correcting data issue

**Status progression:** `open` → `in_progress` → `closed`

**Example:**
```bash
bd create "Fix margin calculation overflow" \
  --type bug \
  --priority 1
```

**Characteristics:**
- Usually standalone
- May reference related epic if part of larger work
- Lightweight path unless bug reveals architectural issue

## Specialized Patterns

### Research Tasks

**Definition:** Task beads created specifically to answer research questions from a spec.

**When to create:**
- `/research` processes spec with "Research Questions" section
- Each question becomes one task bead

**Pattern:**
```bash
# From spec with 3 research questions, create 3 tasks:
bd create "Research: authentication flow" \
  --type task \
  --parent myproject-abc \
  --priority 2

bd create "Research: validation ordering" \
  --type task \
  --parent myproject-abc \
  --priority 2

bd create "Research: error propagation" \
  --type task \
  --parent myproject-abc \
  --priority 2
```

**Key rules:**
- ONE question per task
- Deep research on single question
- Close when question answered
- Do NOT attempt all questions in one session

**Why separate tasks:**
- Deep research produces better documentation than shallow
- Task beads track progress across sessions
- Users can prioritize which questions matter most
- Each research document is focused and actionable

### Implementation Tasks (Phase Tasks)

**Definition:** Tasks created from implementation plan phases.

**When to create:**
- `/taskify` processes plan
- Each phase or logical unit becomes a task

**Pattern:**
```bash
# Plan has 3 phases, create 3 tasks with dependencies:
bd create "Phase 1: Database schema" --type task --parent myproject-abc
bd create "Phase 2: API integration" --type task --parent myproject-abc
bd create "Phase 3: Testing" --type task --parent myproject-abc

# Set dependencies
bd dep add myproject-abc.2 myproject-abc.1  # Phase 2 depends on Phase 1
bd dep add myproject-abc.3 myproject-abc.2  # Phase 3 depends on Phase 2
```

**Characteristics:**
- Map to plan phases
- Have explicit dependencies
- Success criteria from plan
- Independently testable

### Handoff Beads (Deprecated Pattern)

**Definition:** Session continuity mechanism for transferring context.

**Status:** Being replaced by rich bead descriptions.

**Old pattern:**
```bash
bd create "Handoff: user auth phase 2" \
  --type handoff \
  --parent myproject-abc
```

**New pattern:** Use any bead type with rich description following the template in `thoughts/templates/bead-description.md`.

**Why deprecated:**
- Handoff is a property of the description, not the type
- Any bead can carry handoff context
- Rich descriptions are more flexible

## Type Selection Decision Tree

```
Is this a multi-PR, multi-session feature?
    │
    ├── YES → Create EPIC
    │         └── Will spawn child tasks
    │
    └── NO
         │
         Is this fixing broken behavior?
         │
         ├── YES → Create BUG
         │
         └── NO
              │
              Is this part of an epic?
              │
              ├── YES → Create TASK with parent
              │
              └── NO → Create FEATURE (standalone)
```

## Priority Guidelines

| Priority | Meaning | Use For |
|----------|---------|---------|
| 0 (P0) | Critical | Blocking issues, production bugs |
| 1 (P1) | High | Active sprint work, important features |
| 2 (P2) | Medium | Standard work, most tasks |
| 3 (P3) | Low | Nice-to-have, improvements |
| 4 (P4) | Backlog | Future consideration |

**Note:** Use numeric (0-4) or P-notation (P0-P4). Never use "high"/"medium"/"low".

## Naming Conventions

### Epics
- Descriptive feature name
- Example: "Non-liquidatable subaccount capabilities"

### Tasks
- Prefix with phase or category
- Example: "Phase 2: Gateway integration"
- Example: "Research: margin check locations"

### Bugs
- Describe the symptom
- Example: "Fix margin calculation overflow on large positions"

### Features
- Action + target
- Example: "Add retry logic to API client"
