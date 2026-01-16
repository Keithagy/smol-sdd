---
description: Continue working on a handoff bead
---

# Take On

Resume work from a handoff bead. Handoffs contain learnings and next steps from previous sessions.

## Getting Started

1. **If bead ID provided**: Claim it and gather context
2. **If no bead ID**: Find available handoffs:
   ```bash
   bd list --status=open | grep -i handoff
   ```

## Process

### Step 1: Claim and Discover Context

```bash
bd update <bead-id> --status in_progress
```

Then spawn `bd-discoverer` agent:

```
Task tool with subagent_type="bd-discoverer"
Prompt: "Discover context for handoff <bead-id>"
```

The agent returns epic chain, linked documents with line references, and key learnings.

### Step 2: Verify Handoff State

After context discovery, verify:
- Recent changes from handoff still exist in codebase
- No breaking changes since handoff
- Blockers resolved or still present

### Step 3: Present and Confirm

```
Handoff <bead-id> claimed. Current situation:

**Context**: [from bd-discoverer]
**Verified**: [changes present/missing]
**Blockers**: [resolved/pending]

**Next Steps** (from handoff):
1. [step]
2. [step]

Proceed with [first step]?
```

Wait for user confirmation.

### Step 4: Execute

1. Create TodoWrite from Next Steps
2. Begin implementation
3. Reference learnings throughout
4. Create new handoff if ending session incomplete

## Scenarios

| Scenario | Indicator | Action |
|----------|-----------|--------|
| Clean | All changes present, no conflicts | Proceed |
| Diverged | Changes missing or modified | Reconcile, adapt |
| Incomplete | Partial implementations | Complete unfinished first |
| Stale | Major refactoring occurred | Re-run `bd-discoverer` on parent epic |

## Workflow Position

Pairs with `/handoff` for multi-session continuity. See `beads-workflow` skill.
