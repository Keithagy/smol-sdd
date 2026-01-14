# Example: Epic Through Full Ceremony

This example shows an epic progressing through all 15 workflow states.

## Scenario

Feature: Add user authentication to the application

## Phase 1: Creation and Specification

### Create Epic

```bash
$ bd create "User authentication feature" --type epic --priority 1

Created: myproject-abc
Status: open
```

### Run /spec

```bash
# Agent runs /spec myproject-abc
# ... interactive spec clarification ...
# Spec written to thoughts/specs/2025-01-08-user-auth.md

$ bd update myproject-abc --status needs_research
$ bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-user-auth.md"

$ bd show myproject-abc
Status: needs_research
```

## Phase 2: Research

### Run /research

```bash
# Agent runs /research myproject-abc
$ bd update myproject-abc --status researching

# ... parallel research tasks ...
# Research written to thoughts/research/2025-01-08-user-auth.md

$ bd update myproject-abc --status needs_research_review
$ bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-user-auth.md
- Research: thoughts/research/2025-01-08-user-auth.md"

$ bd show myproject-abc
Status: needs_research_review
```

### Human Review

```bash
# Human reviews research document
# Human approves research

$ bd update myproject-abc --status needs_design

$ bd show myproject-abc
Status: needs_design
```

## Phase 3: Design

### Run /techdesign

```bash
# Agent runs /techdesign myproject-abc
$ bd update myproject-abc --status designing

# ... design process ...
# Design written to thoughts/design/2025-01-08-user-auth.md

$ bd update myproject-abc --status needs_design_review
$ bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-user-auth.md
- Research: thoughts/research/2025-01-08-user-auth.md
- Design: thoughts/design/2025-01-08-user-auth.md"

$ bd show myproject-abc
Status: needs_design_review
```

### Human Review

```bash
# Human reviews design document
# Human approves design

$ bd update myproject-abc --status needs_plan

$ bd show myproject-abc
Status: needs_plan
```

## Phase 4: Planning

### Run /plan

```bash
# Agent runs /plan myproject-abc
$ bd update myproject-abc --status planning

# ... planning process ...
# Plan written to thoughts/plans/2025-01-08-myproject-abc-user-auth.md

$ bd update myproject-abc --status needs_plan_review
$ bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-user-auth.md
- Research: thoughts/research/2025-01-08-user-auth.md
- Design: thoughts/design/2025-01-08-user-auth.md
- Plan: thoughts/plans/2025-01-08-myproject-abc-user-auth.md"

$ bd show myproject-abc
Status: needs_plan_review
```

### Human Review

```bash
# Human reviews implementation plan
# Human approves plan

$ bd update myproject-abc --status needs_task_breakdown

$ bd show myproject-abc
Status: needs_task_breakdown
```

## Phase 5: Task Breakdown

### Run /taskify

```bash
# Agent runs /taskify myproject-abc

# Create tasks
$ bd create "Phase 1: Database schema" --type task --parent myproject-abc --priority 2
Created: myproject-abc.1

$ bd create "Phase 2: API integration" --type task --parent myproject-abc --priority 2
Created: myproject-abc.2

$ bd create "Phase 3: Integration tests" --type task --parent myproject-abc --priority 2
Created: myproject-abc.3

# Set dependencies
$ bd dep add myproject-abc.2 myproject-abc.1  # Phase 2 after Phase 1
$ bd dep add myproject-abc.3 myproject-abc.2  # Phase 3 after Phase 2

$ bd update myproject-abc --status needs_implementation

$ bd show myproject-abc
Status: needs_implementation
Children:
  - myproject-abc.1: Phase 1: Database schema [open]
  - myproject-abc.2: Phase 2: API integration [blocked]
  - myproject-abc.3: Phase 3: Integration tests [blocked]
```

## Phase 6: Implementation

### Start Implementation

```bash
# Agent runs /implement myproject-abc.1
$ bd update myproject-abc --status implementing
$ bd update myproject-abc.1 --status in_progress

$ bd show myproject-abc
Status: implementing

$ bd ready
(no tasks shown - myproject-abc.1 is in_progress)
```

### Complete Phase 1

```bash
# ... implementation work ...

$ bd close myproject-abc.1

$ bd ready
myproject-abc.2: Phase 2: API integration [open]
```

### Complete Phase 2

```bash
$ bd update myproject-abc.2 --status in_progress
# ... implementation work ...
$ bd close myproject-abc.2

$ bd ready
myproject-abc.3: Phase 3: Integration tests [open]
```

### Complete Phase 3

```bash
$ bd update myproject-abc.3 --status in_progress
# ... implementation work ...
$ bd close myproject-abc.3

# All tasks complete
$ bd update myproject-abc --status needs_implementation_review

$ bd show myproject-abc
Status: needs_implementation_review
Children:
  - myproject-abc.1: Phase 1: Database schema [closed]
  - myproject-abc.2: Phase 2: API integration [closed]
  - myproject-abc.3: Phase 3: Integration tests [closed]
```

### PR Review

```bash
# Create PR, get review
# PR approved and merged

$ bd update myproject-abc --status testing

$ bd show myproject-abc
Status: testing
```

## Phase 7: Validation

### Run /implement --validate

```bash
# Agent runs /implement myproject-abc --validate

# ... validation checks ...
# All success criteria pass

$ bd close myproject-abc --reason "Validated and tested successfully"

$ bd show myproject-abc
Status: closed
Reason: Validated and tested successfully
```

## Timeline Summary

| State | Duration | Actor |
|-------|----------|-------|
| open | 1 min | System |
| needs_spec | 30 min | Agent |
| needs_research | Immediate | Agent |
| researching | 45 min | Agent |
| needs_research_review | 1 day | Human |
| needs_design | Immediate | Human |
| designing | 1 hour | Agent |
| needs_design_review | 1 day | Human |
| needs_plan | Immediate | Human |
| planning | 1 hour | Agent |
| needs_plan_review | 1 day | Human |
| needs_task_breakdown | Immediate | Human |
| needs_implementation | 15 min | Agent |
| implementing | 2 days | Agent(s) |
| needs_implementation_review | 1 day | Human |
| testing | 30 min | Agent |
| closed | — | — |

## Key Observations

1. **Human gates** slow the process but ensure quality
2. **Agent work** is concentrated in active phases
3. **Status progression** is always forward (no backtracking)
4. **Artifacts accumulate** in the Documents section
5. **Tasks are created late** (after plan approval)
6. **Multiple agents** can work during `implementing` phase
