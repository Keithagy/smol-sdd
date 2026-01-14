---
description: Create a handoff for steered compaction to enable multi-session work
---

# Handoff

You are tasked with creating a handoff bead to transfer your work to another agent in a new session. The handoff is a subtask bead linked to your current working bead.

**Goal**: Compact and summarize your context into a bead description without losing key details.

## Process

### 1. Identify Parent Bead

You should already know your parent bead ID—you claimed it at session start via `/takeon` or `bd update <id> --status in_progress`.

**If you're uncertain which bead is the parent**, ask the user to confirm:
```
I'm about to create a handoff. Which bead should I link this to?
- bead-a3f8: [title]
- bead-b7c2: [title]
```

If you need to check:
```bash
bd list --status=in_progress
```

### 2. Determine What's Next

**If the user specified what's next** ("land the plane, next we will..."):
- Use their direction for the Next Steps section

**If the user didn't specify**:
1. Trace the dependency chain upward:
   ```bash
   bd show <parent-id>
   ```
2. Find associated implementation plan in `thoughts/plans/`
3. Determine the next step from the plan's phase structure
4. If no plan exists, summarize logical next actions based on current progress

### 3. Create Handoff Bead

Create a **subtask** bead as a child of the parent:

```bash
bd create "Handoff: <concise-description>" \
  --parent <parent-id> \
  -p 1 \
  --description "$(cat <<'EOF'
<bead-description-here>
EOF
)"
```

**Required sections in the description:**

```markdown
## Status
[What's done, in progress, blocked - be precise: "Phase 2 of 4, step 3 complete"]

## Critical References
[2-3 key file paths - design, plan, relevant patterns]

## Recent Changes
[Code changes with `file:line` syntax]

## Learnings
[Key discoveries, decisions, what didn't work]

## Artifacts
[Documents produced or updated]

## Blockers
[Current blockers or "None"]

## Next Steps
[Ordered immediate actions]

## Other Notes
[Edge cases, gotchas, tactical info]
```

### 4. Sync and Respond

After creating the handoff bead:

```bash
bd sync
```

Then respond to the user with:

---

Handoff created! Resume in a new session with:

```bash
/takeon <handoff-bead-id>
```

---

## Example

```bash
bd create "Handoff: auth validation phase 2" \
  --parent bead-b7c2 \
  -p 1 \
  --description "$(cat <<'EOF'
## Status
Phase 2 of 3 complete. Validation logic implemented, unit tests passing. Integration tests in progress.

## Critical References
- Plan: `thoughts/plans/2025-01-08-auth-validation.md`
- Pattern: `src/services/auth/validator.ts:42-98`

## Recent Changes
- `src/services/auth/validator.ts:156` — Added token validation
- `src/lib/crypto/jwt.ts:89-102` — New signing helper

## Learnings
- Token refresh requires checking both access and refresh token expiry
- The `TOKEN_EXPIRY_BUFFER` constant (60s) must be used, not hardcoded values
- Existing pattern at `src/services/auth/middleware.ts:234` shows correct flow

## Artifacts
- `thoughts/plans/2025-01-08-auth-validation.md` — Implementation plan (updated)

## Blockers
None

## Next Steps
1. Complete integration tests in `tests/integration/auth/*.test.ts`
2. Run full test suite: `npm test`
3. Begin Phase 3: API integration

## Other Notes
- The auth service emits events on `auth.token.validated` — listener pattern at `src/events/auth.ts:78`
- Deferred: OAuth provider support (out of scope per plan)
EOF
)"
```

Response:
```
Handoff created! Resume in a new session with:

/takeon bead-c9d4
```

## Guidelines

- **Be thorough but concise.** Include both high-level objectives and tactical details.
- **Prefer file:line references over code snippets.** Let the next agent read fresh code.
- **Be precise about status.** "Phase 2 of 4, step 3 complete" beats "in progress."
- **Think about what you wished you knew** when you started this session.

## Workflow Position

**Current stage:** `/handoff` — Transfer context for multi-session work

Handoffs enable continuity across agent sessions without context loss. See the `beads-workflow` skill for how handoffs fit into the epic workflow.
