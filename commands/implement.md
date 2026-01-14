---
description: Complete or verify an implementation task
model: opus
---

# Implement

You are tasked with either **completing** implementation work or **verifying** that completed work meets success criteria. Context is discovered through bead graph traversal, not direct file paths.

## Getting Started

When this command is invoked:

1. **If a bead ID is provided** (e.g., `/implement bead-abc123`):
   - Run `bd show <bead-id>` to read the task bead
   - **Claim the task**: Run `bd update <bead-id> --status in_progress` to mark it as claimed
   - Begin Context Discovery Protocol (see below)
   - Proceed with implementation

2. **If NO bead ID is provided**:
   - Run `bd ready` to show available tasks
   - Present the list to the user and ask which bead to work on
   - If no beads are ready: "No ready tasks available. Use `/taskify` to create tasks from a plan, or create a bead first."
   - Wait for user input before proceeding

**IMPORTANT**: Do NOT ask for file paths to plans or design docs. Context comes from the bead graph.

## Context Discovery Protocol

Gather context through the bead dependency graph — NOT by reading thoughts/ files directly.

1. **Start**: `bd show <task-bead>` — read description and dependencies
2. **Traverse**: Walk dependency graph upward, gathering context from each bead's description
3. **Upon reaching spec epic, assess**: Does progressive disclosure from beads provide sufficient context?
   - **YES** → Proceed with implementation
   - **NO** → Continue to step 4
4. **Discover prereq beads**: Find research/plan/design child beads of the spec epic
5. **Fallback**: Only now read the actual research/plan/design documents linked in those prereq beads' descriptions

### Design Documents as Constitution

When you discover a design document via the bead graph (step 5), treat it as authoritative for:

- **State management**: Where state lives and why
- **Type decisions**: Which fields use which types
- **Failure modes**: How errors should be handled
- **Component responsibilities**: Who owns what behavior
- **API contracts**: Request/response shapes and validation rules

When you encounter tactical decisions during implementation, consult the design document's principles. If the design doesn't cover it, use your judgment aligned with the design's philosophy.

If you discover that implementation cannot follow the design, **STOP** and flag this:

```
Design Conflict Detected:

Design says: [architectural decision from design doc]
Implementation shows: [what you've discovered]
Why it conflicts: [explanation]

Options:
1. Proceed anyway (minor deviation)
2. Request /techdesign to update the design
3. Need human guidance

Recommendation: [your recommendation]
```

### Status Transitions

**On start** (if epic found and status is `needs_implementation`):

```bash
bd update <epic-id> --status implementing
```

**On phase completion**: No epic status change (remains `implementing` until all tasks done)

**When all child tasks under epic are closed**:

```bash
bd update <epic-id> --status needs_implementation_review
```

**Present to user**:

```
All tasks complete. Epic <epic-id> advanced to `needs_implementation_review`.

Please review the implementation and create PR. When approved, run `/validate` to verify.
```

## Implementation Philosophy

Plans are carefully designed, but reality can be messy. Your job is to:

- Follow the plan's intent while adapting to what you find
- Implement each phase fully before moving to the next
- Verify your work makes sense in the broader codebase context
- Update checkboxes in the plan as you complete sections
- Avoid overdocumenting: Are you adding meaningless comments to one-liners that can drift and rot?

When things don't match the plan exactly, think about why and communicate clearly.

If you encounter a mismatch:

- STOP and think deeply about why the plan can't be followed
- Present the issue clearly:

  ```
  Issue in Phase [N]:
  Expected: [what the plan says]
  Found: [actual situation]
  Why this matters: [explanation]

  How should I proceed?
  ```

## Verification Approach

After implementing a phase:

- Run the success criteria checks from the plan
- Fix any issues before proceeding
- Update your progress in both the plan and your todos
- Check off completed items in the plan file itself using Edit
- **Pause for human verification**: After completing all automated verification for a phase:

  ```
  Phase [N] Complete - Ready for Manual Verification

  Automated verification passed:
  - [List automated checks that passed]

  Please perform the manual verification steps listed in the plan:
  - [List manual verification items from the plan]

  Let me know when manual testing is complete so I can proceed to Phase [N+1].
  ```

If instructed to execute multiple phases consecutively, skip the pause until the last phase.

Do not check off items in the manual testing steps until confirmed by the user.

## If You Get Stuck

When something isn't working as expected:

- First, make sure you've read and understood all the relevant code
- Consider if the codebase has evolved since the plan was written
- Consult the **project-patterns skill** (if configured) for implementation patterns
- Present the mismatch clearly and ask for guidance

Use sub-tasks sparingly - mainly for targeted debugging or exploring unfamiliar territory.

## Resuming Work

If the plan has existing checkmarks:

- Trust that completed work is done
- Pick up from the first unchecked item
- Verify previous work only if something seems off

Remember: You're implementing a solution, not just checking boxes.

---

## Verify Mode

When verifying completed work (e.g., `/implement verify bead-abc` or when asked to validate):

### Verification Process

1. **Discover what to verify** by traversing the bead graph:
   - If task bead: Find parent epic
   - If epic bead: List all child task beads
   - Check bead descriptions for plan references

2. **Gather bead state**:
   ```bash
   bd show <bead-id>
   bd list --status=in_progress  # Related active work
   ```

3. **Cross-reference bead state with code state**:
   - Beads marked complete → verify code exists
   - Beads still open → verify work is truly incomplete

4. **Run automated verification** from the plan's success criteria

5. **Generate validation report**:

```markdown
## Validation Report: [Plan Name]

### Implementation Status
✓ Phase 1: [Name] - Fully implemented
✓ Phase 2: [Name] - Fully implemented
⚠️ Phase 3: [Name] - Partially implemented (see issues)

### Automated Verification Results
✓ Build passes
✓ Tests pass
✗ Linting issues: [details]

### Code Review Findings

#### Matches Plan:
- [What was implemented correctly]

#### Deviations from Plan:
- [Any differences and whether they're improvements or issues]

#### Potential Issues:
- [Problems found]

### Manual Testing Required:
1. [ ] [Verification step]
2. [ ] [Another step]

### Recommendations:
- [Actions needed before merge]
```

### Verification Status Transitions

**On validation success**:
```bash
bd update <epic-id> --status testing
```

**On testing complete** (user confirms):
```bash
bd close <epic-id> --reason "Validated and tested successfully"
```

### Beads Cleanup

After successful validation:

1. **Close all task beads** associated with this plan:
   ```bash
   bd close <id1> <id2> ... --reason "Validated: [summary]"
   ```

2. **Update epic bead** to reflect completion

---

## Workflow Position

**Current stage:** `/implement` — Complete or verify work

This is Step 6-7 of the epic workflow. See the `beads-workflow` skill for the complete state machine and context discovery protocol.

## Committing Changes

When committing implementation code, use the **task bead ID** (not the epic):

```bash
git add <changed-files>
git commit -m "[<task-bead-id>] <descriptive message>"
```

Example:
```bash
git commit -m "[bead-xyz] Add margin validation to risk service"
```

**Do NOT include** auto-generated signatures.
