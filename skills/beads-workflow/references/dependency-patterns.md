# Dependency Patterns

This document describes how to use bead dependencies for workflow coordination.

## Dependency Types

### Parent-Child (Containment)

**Purpose:** Hierarchical organization. Epic contains tasks.

**Semantics:**
- Child inherits context from parent
- Child completion contributes to parent progress
- Parent status reflects aggregate child status

**Creation:**
```bash
bd create "Phase 1: Config" --type task --parent myproject-abc
```

**Querying:**
```bash
# List all children of an epic
bd list --parent myproject-abc

# Show parent from child
bd show myproject-abc.1  # Parent field shows myproject-abc
```

### Blocks (Sequencing)

**Purpose:** Enforce ordering between tasks.

**Semantics:**
- Task A blocks Task B → B cannot start until A is closed
- `bd ready` excludes blocked tasks
- Multiple tasks can block a single task (convergence)
- Single task can block multiple tasks (fan-out)

**Creation:**
```bash
# "myproject-xyz depends on myproject-abc"
# (abc must complete before xyz can start)
bd dep add myproject-xyz myproject-abc
```

**Querying:**
```bash
# Show what blocks a task
bd show myproject-xyz  # Blocked-by field

# Show all blocked tasks
bd blocked

# Show ready (unblocked) tasks
bd ready
```

## Common Patterns

### Sequential Phases

For work that must happen in order:

```
Phase 1 → Phase 2 → Phase 3
```

```bash
# Create tasks
bd create "Phase 1: Schema" --type task --parent myproject-abc
bd create "Phase 2: Backend" --type task --parent myproject-abc
bd create "Phase 3: Frontend" --type task --parent myproject-abc

# Set dependencies
bd dep add myproject-abc.2 myproject-abc.1  # Phase 2 after Phase 1
bd dep add myproject-abc.3 myproject-abc.2  # Phase 3 after Phase 2
```

Result:
- `bd ready` shows only Phase 1
- After Phase 1 closes, Phase 2 appears in `bd ready`
- After Phase 2 closes, Phase 3 appears

### Parallel Tracks

For independent work streams:

```
Track A: [A1] → [A2]
Track B: [B1] → [B2]
                 ↘
                  → [Integration]
                 ↗
```

```bash
# Track A
bd create "A1: Backend API" --type task --parent myproject-abc
bd create "A2: Backend tests" --type task --parent myproject-abc
bd dep add myproject-abc.2 myproject-abc.1

# Track B
bd create "B1: Frontend UI" --type task --parent myproject-abc
bd create "B2: Frontend tests" --type task --parent myproject-abc
bd dep add myproject-abc.4 myproject-abc.3

# Integration (depends on both tracks)
bd create "Integration testing" --type task --parent myproject-abc
bd dep add myproject-abc.5 myproject-abc.2  # After A2
bd dep add myproject-abc.5 myproject-abc.4  # After B2
```

Result:
- A1 and B1 both appear in `bd ready` (can run in parallel)
- Multiple agents can claim different tracks
- Integration waits for both tracks to complete

### Convergence Point

When multiple tasks must complete before the next:

```
[Task 1] ─┐
[Task 2] ─┼──→ [Convergence Task]
[Task 3] ─┘
```

```bash
bd create "Task 1" --type task --parent myproject-abc
bd create "Task 2" --type task --parent myproject-abc
bd create "Task 3" --type task --parent myproject-abc
bd create "Convergence" --type task --parent myproject-abc

bd dep add myproject-abc.4 myproject-abc.1
bd dep add myproject-abc.4 myproject-abc.2
bd dep add myproject-abc.4 myproject-abc.3
```

### Fan-Out

When one task enables multiple parallel tasks:

```
              ┌──→ [Task A]
[Setup] ──────┼──→ [Task B]
              └──→ [Task C]
```

```bash
bd create "Setup" --type task --parent myproject-abc
bd create "Task A" --type task --parent myproject-abc
bd create "Task B" --type task --parent myproject-abc
bd create "Task C" --type task --parent myproject-abc

bd dep add myproject-abc.2 myproject-abc.1  # A after Setup
bd dep add myproject-abc.3 myproject-abc.1  # B after Setup
bd dep add myproject-abc.4 myproject-abc.1  # C after Setup
```

## Dependency Graph Visualization

```bash
# View dependency tree from a bead
bd dep tree myproject-abc
```

Example output:
```
myproject-abc (epic)
├── myproject-abc.1: Phase 1: Schema [open]
│   └── blocks: myproject-abc.2
├── myproject-abc.2: Phase 2: Backend [blocked]
│   ├── blocked-by: myproject-abc.1
│   └── blocks: myproject-abc.3
└── myproject-abc.3: Phase 3: Frontend [blocked]
    └── blocked-by: myproject-abc.2
```

## Coordination with `bd ready`

`bd ready` shows tasks that are:
1. Status is `open` (not `in_progress` or `closed`)
2. No blocking dependencies (or all blockers are `closed`)
3. Ready to be claimed

**Workflow for parallel agents:**
```bash
# Agent 1
bd ready                                    # See available tasks
bd update myproject-abc.1 --status in_progress  # Claim task
# ... work ...
bd close myproject-abc.1                     # Release downstream tasks

# Agent 2 (running concurrently)
bd ready                                    # Different tasks available
bd update myproject-abc.3 --status in_progress
# ... work ...
```

## Best Practices

### Do:
- Create explicit dependencies for sequential work
- Use convergence points for integration phases
- Let `bd ready` drive task selection
- Close tasks as soon as complete (unblocks downstream)

### Don't:
- Create circular dependencies (A blocks B, B blocks A)
- Over-specify dependencies (implicit ordering is OK sometimes)
- Leave tasks `in_progress` when done (blocks downstream)
- Forget to close research/prereq tasks

## Debugging Blocked Tasks

```bash
# Find why a task is blocked
bd show myproject-xyz
# Look for "blocked-by" in output

# Check status of blocker
bd show <blocker-id>

# If blocker should be done, close it
bd close <blocker-id>
```

## Removing Dependencies

If a dependency was created incorrectly:

```bash
# Remove dependency (xyz no longer depends on abc)
bd dep remove myproject-xyz myproject-abc
```
