# Context Discovery Protocol

This document describes the algorithm for discovering context through the bead graph instead of direct file access.

## Core Principle

**Bead graph is the primary context source, not file paths.**

The paradigm shift: Instead of agents reading files directly based on paths, they traverse the bead dependency graph to discover what context they need. This enables:

- Progressive disclosure (load only what's needed)
- Self-contained beads (work with just descriptions)
- Parallel coordination (multiple agents via `bd ready`)
- Session continuity (context persists in beads)

## The Algorithm

### Step 1: Start with Task Bead

```bash
bd show <task-bead-id>
```

Extract from the bead:
- Title and description
- Parent relationship
- Dependencies (blocks/blocked-by)
- Status

**Assess**: Is this enough context to proceed?
- **YES** → Begin work
- **NO** → Continue to Step 2

### Step 2: Traverse Upward

Follow parent relationships:

```bash
bd show <task-bead-id>  # Find parent field
bd show <parent-id>     # Read parent bead
```

For each parent bead:
1. Read its description
2. Extract relevant context
3. Note any artifact links (design, plan, research)

Continue traversing until reaching the **spec epic** (root of the work).

### Step 3: Assess at Spec Epic

Upon reaching the spec epic, evaluate:

**Sufficient context indicators:**
- Clear understanding of the task scope
- Know which files to modify
- Understand the patterns to follow
- Know the success criteria

**Insufficient context indicators:**
- Unsure about implementation approach
- Need to understand existing code patterns
- Require detailed design decisions
- Need historical context or learnings

If sufficient → Proceed with implementation
If insufficient → Continue to Step 4

### Step 4: Find Prerequisite Beads

Look for child beads of the epic that represent completed prerequisite work:

```bash
bd list --parent <epic-id>
```

Common prerequisite bead types:
- **Research beads** — Document codebase findings
- **Design beads** — Capture architectural decisions
- **Plan beads** — Define implementation phases

These beads contain learnings and decisions from previous sessions.

### Step 5: Read Linked Documents (Fallback)

Only now, if still insufficient, read the actual documents linked in bead descriptions:

```markdown
## Documents
- Spec: thoughts/specs/YYYY-MM-DD-feature.md
- Research: thoughts/research/YYYY-MM-DD-feature.md
- Design: thoughts/design/YYYY-MM-DD-feature.md
- Plan: thoughts/plans/YYYY-MM-DD-feature.md
```

Read these FULLY (no limit/offset) when they contain critical context.

## Decision Tree

```
Start: bd show <task-bead>
         │
         ▼
┌────────────────────┐
│ Enough context?    │
└────────┬───────────┘
         │
    ┌────┴────┐
    │         │
   YES        NO
    │         │
    ▼         ▼
 [Work]   Traverse to parent
              │
              ▼
         bd show <parent>
              │
              ▼
┌────────────────────┐
│ At spec epic?      │
└────────┬───────────┘
         │
    ┌────┴────┐
    │         │
    NO       YES
    │         │
    ▼         ▼
 [Loop]  ┌────────────────────┐
         │ Enough context?    │
         └────────┬───────────┘
                  │
             ┌────┴────┐
             │         │
            YES        NO
             │         │
             ▼         ▼
          [Work]   Find prereq beads
                       │
                       ▼
                  bd list --parent <epic>
                       │
                       ▼
              Read prereq descriptions
                       │
                       ▼
         ┌────────────────────┐
         │ Enough context?    │
         └────────┬───────────┘
                  │
             ┌────┴────┐
             │         │
            YES        NO
             │         │
             ▼         ▼
          [Work]   Read linked documents
                       │
                       ▼
                    [Work]
```

## Practical Example

### Scenario: Implementing Phase 2 Task

```bash
# Step 1: Start with task
$ bd show myproject-xyz

Title: Phase 2: Add API validation
Parent: myproject-abc
Status: open
Description:
  Implement validation in API handlers per plan.
  Blocks: myproject-xyz2 (Phase 3)

# Step 2: Check parent (still a task, continue up)
$ bd show myproject-abc

Title: User authentication feature
Type: epic
Status: implementing
Description:
  ## Documents
  - Plan: thoughts/plans/2025-01-08-user-auth.md
  - Design: thoughts/design/2025-01-08-user-auth.md

# Step 3: At epic, assess context
# - Know it's Phase 2 of user auth
# - Know there's a plan and design
# - But don't know the specific patterns yet

# Step 4: Find prereq beads
$ bd list --parent myproject-abc

myproject-abc.1: Research: auth flow locations (closed)
myproject-abc.2: Design review (closed)
myproject-xyz: Phase 2: Add API validation (open) ← current

# Read research bead
$ bd show myproject-abc.1

Description:
  ## Learnings
  - Token validation in src/middleware/auth.ts:156
  - Session handling in src/services/session.ts:234
  - Use TOKEN_EXPIRY_SECONDS constant, not hardcoded

# Step 5: Still need detailed phase steps
# Read the plan document
$ cat thoughts/plans/2025-01-08-user-auth.md
# [Full plan with Phase 2 details]
```

## Anti-Patterns

### Anti-Pattern 1: Direct File Access First

**Wrong:**
```
1. Read thoughts/plans/feature.md
2. Read thoughts/design/feature.md
3. Start implementing
```

**Right:**
```
1. bd show <task-bead>
2. Traverse to epic
3. Read prereq bead descriptions
4. Only then read documents if needed
```

### Anti-Pattern 2: Ignoring Bead Descriptions

**Wrong:**
```
1. bd show <task-bead>  # Just get the ID
2. Immediately read plan file
```

**Right:**
```
1. bd show <task-bead>  # Read FULL description
2. Extract learnings from description
3. Only read files if description insufficient
```

### Anti-Pattern 3: Skipping Prerequisite Beads

**Wrong:**
```
1. bd show <task-bead>
2. Jump straight to plan document
```

**Right:**
```
1. bd show <task-bead>
2. bd list --parent <epic>  # Find prereqs
3. Read prereq descriptions for learnings
4. Then read documents if needed
```

## Benefits of This Protocol

1. **Progressive disclosure** — Context loaded incrementally as needed
2. **Session continuity** — Learnings persist in bead descriptions
3. **Parallel safety** — Multiple agents can discover independently
4. **Reduced context bloat** — Don't load entire documents upfront
5. **Explicit dependencies** — Bead graph shows what relates to what
