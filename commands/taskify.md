---
description: Create or iterate on the breakdown of implementation tasks for some plan
model: opus
---

# Task Breakdown

You are tasked with either **creating** a new task breakdown from a plan or **iterating** on an existing task graph based on feedback or discoveries.

## Mode Detection

When this command is invoked, determine the mode:

1. **Create Mode**: Plan file provided with no existing task beads
2. **Iterate Mode**: Existing task beads need adjustment (split, merge, add, re-prioritize)

---

## ITERATE MODE

When refining an existing task breakdown:

### Step 1: Check Current State

```bash
bd list
bd dep tree <epic-id>  # if epic exists
```

Present current state:

```
Current task breakdown:

[output of bd dep tree or bd list]

What would you like to adjust?
- Split a task into subtasks
- Merge tasks that are too granular
- Add discovered work
- Adjust dependencies
- Re-prioritize tasks
```

### Common Operations

#### Split Task

When a task is too large or reveals multiple distinct work items:

```bash
# Create subtasks
bd create "Subtask 1" -p 1 -t task
bd create "Subtask 2" -p 1 -t task

# Add as children of original
bd dep add bead-subtask1 bead-original
bd dep add bead-subtask2 bead-original

# Update original to be a parent/epic
bd update bead-original -t epic
```

#### Merge Tasks

When tasks are too granular:

```bash
# Close the granular tasks
bd close bead-task1 bead-task2 --reason="Merged into bead-combined"

# Create combined task
bd create "Combined: [description]" -p 1 -t task

# Set up dependencies
bd dep add bead-combined bead-parent
```

#### Add Discovered Work

When implementation reveals new required tasks:

```bash
# Create new task with context
bd create "Discovered: [task description]" -p 2 -t task

# Add to description explaining discovery
bd update bead-new --description "## Discovery Context
Found while working on bead-current: [explanation]

## Scope
- [what needs to be done]

## Success Criteria
- [ ] [testable criterion]
"

# Add appropriate dependencies
bd dep add bead-new bead-prerequisite
```

#### Adjust Dependencies

```bash
# Remove incorrect dependency
bd dep remove bead-child bead-wrong-parent

# Add correct dependency
bd dep add bead-child bead-correct-parent
```

#### Re-prioritize

```bash
# Update priority (0 = highest, 4 = backlog)
bd update bead-XXXX -p 0
```

### After Iteration

Present the updated state:

```
Task breakdown updated. New state:

bd dep tree bead-epic

[output]

**Ready tasks**:
[list of tasks with no blockers]

**Blocked tasks**:
[list with their blockers]

Proceed with `/implement` or continue refining?
```

### Iterate Mode Guidelines

1. **Preserve Context**: When splitting, copy relevant context to subtasks
2. **Update Parent**: Mark split tasks as epics if they now have children
3. **Verify Graph**: Run `bd dep tree` after changes to verify structure
4. **Link Discoveries**: Always reference the task where discovery was made
5. **Sync Changes**: After significant changes, run `bd sync` to persist

---

## CREATE MODE

When creating a new task breakdown from a plan:

### Initial Response

When this command is invoked:

1. **If a plan file path is provided**:
   - Read the plan file FULLY
   - Analyze phases and their relationships
   - Begin task breakdown process

2. **If no plan provided**, respond with:

   ```
   I'll help you break down an implementation plan into beads tasks.

   Please provide:
   1. The plan file path or epic bead ID (e.g., thoughts/plans/2025-01-08-bead-abc-feature.md or bead-abc)

   I'll analyze the phases and create a dependency-aware task graph in beads.
   ```

## Process Steps

### Step 1: Analyze Plan Structure

Read the plan and identify:

1. **Phases**: Major implementation stages
2. **Dependencies**: Which phases depend on others
3. **Parallelizable work**: What can be done concurrently
4. **Testable units**: Natural boundaries for tasks

Present analysis:

```
Based on the plan, I see this structure:

**Phases:**
1. [Phase name] - [what it accomplishes]
   - Depends on: [nothing / Phase X]
   - Parallelizable: [yes/no]

2. [Phase name] - [what it accomplishes]
   - Depends on: Phase 1
   - Parallelizable: [yes/no]

**Proposed Task Breakdown:**
- Epic: [overall feature]
  - Task 1.1: [first subtask of phase 1]
  - Task 1.2: [second subtask, can parallel with 1.1]
  - Task 2.1: [first subtask of phase 2, blocks on 1.1 + 1.2]

**Parallel Execution Overview:**
```
Timeline ->
-------------------------------------------------------------
Track A:  [Task 1.1: Backend API] -----------+
                                             +---> [Task 2.1: Integration]
Track B:  [Task 1.2: Frontend UI] -----------+

Track C:  [Task 1.3: Tests] ----------------------> [Task 2.2: E2E Tests]
-------------------------------------------------------------
```

- **Independent tracks** (A, B, C) can run concurrently in separate worktrees
- **Convergence points** (Task 2.1) wait for all upstream dependencies
- **Parallel agents** can claim different ready tasks simultaneously via `bd ready`

Does this breakdown make sense? Should any tasks be split further or combined?
```

### Step 2: Get Approval on Structure

Wait for user confirmation before creating beads.

### Step 3: Create Beads

After approval, create the task graph:

```bash
# Create epic bead (if not already exists)
bd create "[Feature Name]" -p 0 -t epic
# Returns: bead-XXXX

# Create phase 1 tasks
bd create "Phase 1.1: [task name]" -p 1 -t task
bd create "Phase 1.2: [task name]" -p 1 -t task

# Create phase 2 tasks
bd create "Phase 2.1: [task name]" -p 2 -t task

# Set up dependencies
bd dep add bead-phase1-1 bead-epic
bd dep add bead-phase1-2 bead-epic
bd dep add bead-phase2-1 bead-phase1-1
bd dep add bead-phase2-1 bead-phase1-2
```

### Step 4: Add Context to Beads

For each bead, update description with:

```bash
bd update bead-XXXX --description "## Context
This task implements [section] of the plan.

## Plan Reference
See: thoughts/plans/[plan-file].md, Phase [N]

## Scope
- [specific deliverable 1]
- [specific deliverable 2]

## Success Criteria
- [ ] [testable criterion 1]
- [ ] [testable criterion 2]

## Dependencies
- Blocks on: [bead IDs]
- Blocked by: [bead IDs]
"
```

### Step 5: Present Task Graph

```
Task breakdown complete. Here's the beads graph:

bd dep tree bead-epic

[output of dependency tree]

**Ready tasks** (can start now):
- bead-XXXX: [task name]
- bead-YYYY: [task name]

**Blocked tasks** (waiting on dependencies):
- bead-ZZZZ: [task name] -- waiting on bead-XXXX

Next step: Run `/implement` and use `bd ready` to select tasks.
```

## Epic Integration

### Finding the Epic

When invoked:

1. **If epic bead ID provided**: Use that epic
2. **If plan file provided**: Find epic referencing that plan via description search
3. **Check for epics in task breakdown phase**: `bd list --type=epic --status=needs_task_breakdown`
4. **Otherwise**: Create new epic from plan

### Status Transitions

**On start** (if epic found and status is `needs_task_breakdown`):
- No status change yet (breakdown in progress)

**On completion** (all task beads created with dependencies):

```bash
bd update <epic-id> --status needs_implementation
```

**Update epic description** with task summary:

```
## Tasks Created
- bead-abc.1: [task 1]
- bead-abc.2: [task 2]
- bead-abc.3: [task 3]

Ready tasks: bead-abc.1, bead-abc.2
```

**Present to user**:

```
Task breakdown complete. Epic <epic-id> advanced to `needs_implementation`.

Run `/implement` and use `bd ready` to select tasks.
```

## Important Guidelines

1. **Testable Units**: Each task should be independently testable
2. **Clear Dependencies**: Explicit dependency graph, no implicit ordering
3. **Rich Context**: Bead descriptions should enable autonomous implementation
4. **Reasonable Granularity**: Not too coarse (multi-day), not too fine (trivial)
5. **Plan Alignment**: Every task traces back to a plan section

## Workflow Position

**Current stage:** `/taskify` — "What are the beads tasks?" (granular)

This is Step 5 of the epic workflow. See the `beads-workflow` skill for the complete state machine, dependency patterns, and task graph examples.

## Committing Changes

Task breakdown primarily creates beads (synced via `bd sync`). If you update any plan or artifact files, commit with the **epic bead ID**:

```bash
git add thoughts/plans/<updated-plan>.md
git commit -m "[<epic-id>] Update plan with task breakdown"
```

**Do NOT include** auto-generated signatures.
