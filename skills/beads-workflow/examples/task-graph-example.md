# Example: Dependency-Aware Task Graph

This example shows how to create a task graph with dependencies for parallel execution.

## Scenario

Feature: Add real-time notifications with three parallel tracks plus integration.

## Plan Structure

```
Phase 1: Foundation (sequential)
  1.1: Add configuration settings
  1.2: Define data models

Phase 2: Parallel Tracks
  Track A: Backend implementation
    2A.1: Add notification service
    2A.2: Implement event handlers

  Track B: Frontend implementation
    2B.1: Add notification store
    2B.2: Implement UI components

  Track C: Testing
    2C.1: Add E2E test scenarios
    2C.2: Add unit tests

Phase 3: Integration (convergence)
  3.1: End-to-end integration tests
```

## Creating the Task Graph

### Step 1: Create All Tasks

```bash
# Foundation tasks (sequential)
bd create "1.1: Add configuration settings" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.1

bd create "1.2: Define data models" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.2

# Track A: Backend
bd create "2A.1: Add notification service" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.3

bd create "2A.2: Implement event handlers" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.4

# Track B: Frontend
bd create "2B.1: Add notification store" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.5

bd create "2B.2: Implement UI components" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.6

# Track C: Testing
bd create "2C.1: Add E2E test scenarios" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.7

bd create "2C.2: Add unit tests" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.8

# Integration
bd create "3.1: End-to-end integration tests" --type task --parent myproject-xyz --priority 2
# Created: myproject-xyz.9
```

### Step 2: Set Dependencies

```bash
# Foundation: 1.2 depends on 1.1
bd dep add myproject-xyz.2 myproject-xyz.1

# Track A: 2A.1 depends on 1.2 (foundation complete)
bd dep add myproject-xyz.3 myproject-xyz.2
# Track A: 2A.2 depends on 2A.1
bd dep add myproject-xyz.4 myproject-xyz.3

# Track B: 2B.1 depends on 1.2 (foundation complete)
bd dep add myproject-xyz.5 myproject-xyz.2
# Track B: 2B.2 depends on 2B.1
bd dep add myproject-xyz.6 myproject-xyz.5

# Track C: 2C.1 depends on 1.2 (foundation complete)
bd dep add myproject-xyz.7 myproject-xyz.2
# Track C: 2C.2 depends on 2C.1
bd dep add myproject-xyz.8 myproject-xyz.7

# Integration: depends on ALL tracks completing
bd dep add myproject-xyz.9 myproject-xyz.4  # After Track A
bd dep add myproject-xyz.9 myproject-xyz.6  # After Track B
bd dep add myproject-xyz.9 myproject-xyz.8  # After Track C
```

### Step 3: Verify Graph

```bash
$ bd dep tree myproject-xyz

myproject-xyz (epic)
├── myproject-xyz.1: 1.1: Add configuration settings [open]
│   └── blocks: myproject-xyz.2
├── myproject-xyz.2: 1.2: Define data models [blocked]
│   ├── blocked-by: myproject-xyz.1
│   └── blocks: myproject-xyz.3, myproject-xyz.5, myproject-xyz.7
├── myproject-xyz.3: 2A.1: Add notification service [blocked]
│   ├── blocked-by: myproject-xyz.2
│   └── blocks: myproject-xyz.4
├── myproject-xyz.4: 2A.2: Implement event handlers [blocked]
│   ├── blocked-by: myproject-xyz.3
│   └── blocks: myproject-xyz.9
├── myproject-xyz.5: 2B.1: Add notification store [blocked]
│   ├── blocked-by: myproject-xyz.2
│   └── blocks: myproject-xyz.6
├── myproject-xyz.6: 2B.2: Implement UI components [blocked]
│   ├── blocked-by: myproject-xyz.5
│   └── blocks: myproject-xyz.9
├── myproject-xyz.7: 2C.1: Add E2E scenarios [blocked]
│   ├── blocked-by: myproject-xyz.2
│   └── blocks: myproject-xyz.8
├── myproject-xyz.8: 2C.2: Add unit tests [blocked]
│   ├── blocked-by: myproject-xyz.7
│   └── blocks: myproject-xyz.9
└── myproject-xyz.9: 3.1: Integration tests [blocked]
    └── blocked-by: myproject-xyz.4, myproject-xyz.6, myproject-xyz.8
```

## Execution Flow

### Initial State

```bash
$ bd ready
myproject-xyz.1: 1.1: Add configuration settings [open]
```

Only Task 1.1 is available.

### After Foundation Complete

```bash
# Complete 1.1
$ bd close myproject-xyz.1

$ bd ready
myproject-xyz.2: 1.2: Define data models [open]

# Complete 1.2
$ bd close myproject-xyz.2

$ bd ready
myproject-xyz.3: 2A.1: Add notification service [open]
myproject-xyz.5: 2B.1: Add notification store [open]
myproject-xyz.7: 2C.1: Add E2E scenarios [open]
```

Three tasks now available in parallel!

### Parallel Execution

```bash
# Agent 1 claims Track A
$ bd update myproject-xyz.3 --status in_progress

# Agent 2 claims Track B
$ bd update myproject-xyz.5 --status in_progress

# Agent 3 claims Track C
$ bd update myproject-xyz.7 --status in_progress

$ bd ready
(empty - all available tasks are claimed)
```

### Track Completion

```bash
# Agent 1 completes 2A.1
$ bd close myproject-xyz.3

$ bd ready
myproject-xyz.4: 2A.2: Implement event handlers [open]

# Agent 1 continues with 2A.2
$ bd update myproject-xyz.4 --status in_progress

# Similarly for other tracks...
```

### Convergence

```bash
# After all tracks complete
$ bd close myproject-xyz.4
$ bd close myproject-xyz.6
$ bd close myproject-xyz.8

$ bd ready
myproject-xyz.9: 3.1: Integration tests [open]
```

Integration test only appears when ALL prerequisite tracks are done.

## Visual Timeline

```
Time →
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Agent 1: [1.1: Config]──[1.2: Models]──┐
                                       │
                                       ├──[2A.1: Service]──[2A.2: Handlers]──┐
                                       │                                      │
Agent 2:                               ├──[2B.1: Store]──[2B.2: UI]───────────┼──[3.1: E2E]
                                       │                                      │
Agent 3:                               └──[2C.1: E2E]──[2C.2: Unit]───────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        Foundation           Parallel Tracks            Convergence
```

## Key Benefits

1. **Parallel execution**: 3 agents can work simultaneously during Phase 2
2. **Clear dependencies**: No guessing about ordering
3. **Automatic unblocking**: `bd ready` shows what's available
4. **Convergence safety**: Integration only runs when all prereqs complete
5. **Progress visibility**: `bd blocked` shows what's waiting

## Anti-Patterns to Avoid

### Don't: Create circular dependencies

```bash
# WRONG: A depends on B, B depends on A
bd dep add myproject-xyz.3 myproject-xyz.4
bd dep add myproject-xyz.4 myproject-xyz.3  # Error!
```

### Don't: Over-specify dependencies

```bash
# WRONG: If 1 → 2 → 3, don't also add 1 → 3
bd dep add myproject-xyz.2 myproject-xyz.1  # Good
bd dep add myproject-xyz.3 myproject-xyz.2  # Good
bd dep add myproject-xyz.3 myproject-xyz.1  # Unnecessary (implicit via 2)
```

### Don't: Forget convergence dependencies

```bash
# WRONG: Integration only waits for one track
bd dep add myproject-xyz.9 myproject-xyz.4  # Only Track A

# RIGHT: Integration waits for ALL tracks
bd dep add myproject-xyz.9 myproject-xyz.4  # Track A
bd dep add myproject-xyz.9 myproject-xyz.6  # Track B
bd dep add myproject-xyz.9 myproject-xyz.8  # Track C
```
