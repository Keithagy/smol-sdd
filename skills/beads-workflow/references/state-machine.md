# Epic Workflow State Machine

This document provides the complete 15-state workflow for epic beads requiring full ceremony.

## State Definitions

### Phase 1: Specification

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `open` | Epic created, no work started | `bd create --type epic` | — |
| `needs_spec` | Awaiting specification document | Manual or automatic after creation | `/spec` |

### Phase 2: Research

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `needs_research` | Spec complete, research needed | `/spec` completes | `/research` |
| `researching` | Agent actively researching | `/research` starts | — |
| `needs_research_review` | Research complete, awaiting approval | `/research` completes | **Human approves** |

### Phase 3: Design

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `needs_design` | Research approved, design needed | Human approves research | `/techdesign` |
| `designing` | Agent creating technical design | `/techdesign` starts | — |
| `needs_design_review` | Design complete, awaiting approval | `/techdesign` completes | **Human approves** |

### Phase 4: Planning

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `needs_plan` | Design approved, plan needed | Human approves design | `/plan` |
| `planning` | Agent creating implementation plan | `/plan` starts | — |
| `needs_plan_review` | Plan complete, awaiting approval | `/plan` completes | **Human approves** |

### Phase 5: Task Breakdown

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `needs_task_breakdown` | Plan approved, tasks needed | Human approves plan | `/taskify` |

### Phase 6: Implementation

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `needs_implementation` | Tasks created, ready to implement | `/taskify` completes | `/implement` |
| `implementing` | Agents working on tasks | `/implement` starts | — |
| `needs_implementation_review` | All tasks done, PR ready | All child tasks closed | **PR approved** |

### Phase 7: Validation

| State | Description | Entry Condition | Exit Action |
|-------|-------------|-----------------|-------------|
| `testing` | PR merged, validation in progress | PR approved and merged | `/implement --validate` |
| `closed` | Epic complete | `/implement --validate` passes | — |

## State Diagram

```
                    ┌─────────────────────────────────────────────────────────────┐
                    │                                                             │
                    ▼                                                             │
┌──────┐    ┌────────────┐    ┌────────────────┐    ┌─────────────┐    ┌─────────────────────┐
│ open │───▶│ needs_spec │───▶│ needs_research │───▶│ researching │───▶│ needs_research_     │
└──────┘    └────────────┘    └────────────────┘    └─────────────┘    │ review              │
               /spec              /research (start)                    └──────────┬──────────┘
                                                                                  │
                    ┌─────────────────────────────────────────────────────────────┘
                    │ Human approves
                    ▼
┌──────────────┐    ┌───────────┐    ┌─────────────────────┐
│ needs_design │───▶│ designing │───▶│ needs_design_review │
└──────────────┘    └───────────┘    └──────────┬──────────┘
   /techdesign (start)                          │
                                                │ Human approves
                    ┌───────────────────────────┘
                    ▼
┌────────────┐    ┌──────────┐    ┌───────────────────┐
│ needs_plan │───▶│ planning │───▶│ needs_plan_review │
└────────────┘    └──────────┘    └─────────┬─────────┘
   /plan (start)                            │
                                            │ Human approves
                    ┌───────────────────────┘
                    ▼
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────┐
│ needs_task_breakdown │───▶│ needs_implementation │───▶│ implementing │
└──────────────────────┘    └──────────────────────┘    └──────┬───────┘
   /taskify                     /implement (start)             │
                                                               │ All tasks closed
                    ┌──────────────────────────────────────────┘
                    ▼
┌─────────────────────────────┐    ┌─────────┐    ┌────────┐
│ needs_implementation_review │───▶│ testing │───▶│ closed │
└─────────────────────────────┘    └─────────┘    └────────┘
         PR approved                /implement --validate
```

## Transition Rules

### Automatic Transitions (Command-Driven)

Commands automatically update epic status:

```bash
# /spec completion
bd update <epic-id> --status needs_research

# /research start
bd update <epic-id> --status researching

# /research completion
bd update <epic-id> --status needs_research_review

# /techdesign start
bd update <epic-id> --status designing

# /techdesign completion
bd update <epic-id> --status needs_design_review

# /plan start
bd update <epic-id> --status planning

# /plan completion
bd update <epic-id> --status needs_plan_review

# /taskify completion
bd update <epic-id> --status needs_implementation

# /implement start (first task claimed)
bd update <epic-id> --status implementing

# When all child tasks closed
bd update <epic-id> --status needs_implementation_review

# /implement --validate start
bd update <epic-id> --status testing

# /implement --validate completion
bd close <epic-id>
```

### Manual Transitions (Human-Driven)

These gates require explicit human action:

```bash
# After reviewing research document
bd update <epic-id> --status needs_design

# After reviewing design document
bd update <epic-id> --status needs_plan

# After reviewing implementation plan
bd update <epic-id> --status needs_task_breakdown

# After PR is approved and merged
bd update <epic-id> --status testing
```

## Checking Current State

```bash
# View epic with current status
bd show <epic-id>

# List epics by status
bd list --type=epic --status=needs_plan

# Find epics awaiting human review
bd list --type=epic --status=needs_research_review
bd list --type=epic --status=needs_design_review
bd list --type=epic --status=needs_plan_review
bd list --type=epic --status=needs_implementation_review
```

## Configuration

Custom statuses are configured via:

```bash
bd config set status.custom "needs_spec,needs_research,researching,needs_research_review,needs_design,designing,needs_design_review,needs_plan,planning,needs_plan_review,needs_task_breakdown,needs_implementation,implementing,needs_implementation_review,testing"
```

## Skipping States

For simpler work, states can be skipped:

- **Skip spec**: Start at `needs_research` if requirements are clear
- **Skip research**: Start at `needs_design` if codebase is understood
- **Skip design**: Start at `needs_plan` for straightforward features
- **Skip full ceremony**: Use lightweight path (`open` → `in_progress` → `closed`)

When skipping, manually set the starting status:

```bash
bd update <epic-id> --status needs_plan
```
