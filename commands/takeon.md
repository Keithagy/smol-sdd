---
description: Continue working on a handoff bead
---

# Take On

You are tasked with resuming work from a handoff bead through an interactive process. Handoff beads contain critical context, learnings, and next steps from previous sessions.

## Initial Response

When this command is invoked:

1. **If a bead ID was provided** (e.g., `bead-a3f8`):
   - Read the handoff bead fully
   - Trace the dependency chain for broader context
   - Read any referenced documents (plans, designs, research)
   - Present analysis and propose next actions

2. **If no parameters provided**, respond with:

   ```
   I'll help you resume work from a handoff. Let me find available handoffs.
   ```

   Then list recent handoffs:

   ```bash
   bd list --status=open | grep -i handoff
   ```

   Present options and wait for user selection.

## Process Steps

### Step 1: Claim the Handoff Bead

**Immediately claim the handoff** so it's marked as your active work:

```bash
bd update <bead-id> --status in_progress
```

### Step 2: Read Handoff Bead

```bash
bd show <bead-id>
```

Extract all sections from the description:

- Status
- Critical References
- Recent Changes
- Learnings
- Artifacts
- Blockers
- Next Steps
- Other Notes

### Step 3: Trace Dependency Chain

Build context by tracing upward:

```bash
bd show <bead-id>  # Check parent bead
```

For each related bead:

1. Read its description for broader context
2. Identify associated plans/designs in `thoughts/plans/` or `thoughts/design/`
3. Understand where this handoff fits in the larger work

### Step 4: Read Referenced Documents

**Do NOT use sub-agents for these critical reads.**

1. Read files from "Critical References" section
2. Read files from "Artifacts" section
3. Read implementation plans mentioned
4. Verify "Recent Changes" still exist in codebase

### Step 5: Synthesize and Present Analysis

Present to the user:

```
I've analyzed the handoff bead <bead-id>. Here's the current situation:

**Parent Context:**
- [Parent bead]: [What it represents]
- [Associated plan]: [Current phase]

**Status from Handoff:**
[Status section content]

**Key Learnings:**
- [Learning 1] — [Still valid / Changed]
- [Learning 2] — [Still valid / Changed]

**Recent Changes Verified:**
- [Change 1] — [Present / Missing / Modified]
- [Change 2] — [Present / Missing / Modified]

**Blockers:**
[Blockers section or "None"]

**Recommended Next Actions:**
Based on the handoff's Next Steps:
1. [Most logical next step]
2. [Second priority action]
3. [Additional tasks if discovered]

**Potential Issues:**
- [Any conflicts or regressions found]
- [Stale references or missing files]

Shall I proceed with [recommended action 1], or would you like to adjust?
```

### Step 6: Get Confirmation

Wait for user approval before proceeding.

### Step 7: Create Action Plan

1. **Use TodoWrite** to create task list from Next Steps

2. **Present the plan:**

   ```
   I've created a task list based on the handoff:

   [Todo list]

   Ready to begin with: [first task]?
   ```

### Step 8: Begin Implementation

1. Start with the first approved task
2. Reference learnings from handoff throughout
3. Apply documented patterns and approaches
4. Update progress as tasks complete

## Guidelines

### Be Thorough in Analysis

- Read the entire handoff bead description
- Verify ALL mentioned changes still exist
- Check for regressions or conflicts
- Read all referenced artifacts

### Be Interactive

- Present findings before starting work
- Get buy-in on the approach
- Allow for course corrections
- Adapt based on current state vs handoff state

### Leverage Handoff Wisdom

- Pay special attention to "Learnings" section
- Apply documented patterns and approaches
- Avoid repeating mistakes mentioned
- Build on discovered solutions
- Check "Other Notes" for tactical info

### Validate Before Acting

- Never assume handoff state matches current state
- Verify file references still exist
- Check for breaking changes since handoff
- Confirm patterns are still valid

### Track Continuity

- Use TodoWrite to maintain task continuity
- Reference the handoff bead in commits
- Document deviations from original plan
- Create a new handoff when ending session

## Common Scenarios

### Scenario 1: Clean Continuation

- All changes from handoff are present
- No conflicts or regressions
- Clear next steps
- **Action**: Proceed with recommended actions

### Scenario 2: Diverged Codebase

- Some changes missing or modified
- New related code added since handoff
- **Action**: Reconcile differences, adapt plan

### Scenario 3: Incomplete Handoff Work

- Tasks marked as "in_progress" in handoff
- Partial implementations exist
- **Action**: Complete unfinished work first

### Scenario 4: Stale Handoff

- Significant time has passed
- Major refactoring has occurred
- **Action**: Re-evaluate strategy, may need fresh research

## Example Interaction

```
User: /takeon bead-c9d4
Assistant: Let me claim and analyze that handoff bead...

[Runs bd update bead-c9d4 --status in_progress]
[Runs bd show bead-c9d4]
[Traces parent chain to bead-b7c2]
[Reads thoughts/plans/2025-01-08-auth-validation.md]
[Verifies recent changes exist in codebase]

I've claimed and analyzed the handoff bead bead-c9d4. Here's the current situation:

**Parent Context:**
- bead-b7c2: Auth validation feature
- Plan: thoughts/plans/2025-01-08-auth-validation.md (Phase 2 of 3)

**Status from Handoff:**
Phase 2 complete. Validation logic implemented, unit tests passing.

**Key Learnings:**
- Token refresh validation pattern — Still valid
- TOKEN_EXPIRY_BUFFER constant usage — Still valid

**Recent Changes Verified:**
- src/services/auth/validator.ts:156 — Present
- src/lib/crypto/jwt.ts:89-102 — Present

**Blockers:**
None

**Recommended Next Actions:**
1. Complete integration tests
2. Run full test suite
3. Begin Phase 3: API integration

Shall I proceed with integration tests, or adjust the approach?

User: Yes, proceed with integration tests
Assistant: I'll create a task list and begin with the integration tests.

[Creates TodoWrite with tasks from Next Steps]

Task list created:
1. Complete integration tests in tests/integration/auth/*.test.ts
2. Run full test suite
3. Begin Phase 3: API integration

Starting with: Complete integration tests...

[Begins implementation work]
```

## Workflow Position

**Current stage:** `/takeon` — Resume multi-session work

This command pairs with `/handoff` to enable work continuity across agent sessions. See the `beads-workflow` skill for how handoffs integrate into the epic workflow.
