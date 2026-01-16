---
name: beads-workflow
description: This skill should be used when working with beads issue tracking, navigating epic workflow states, discovering context from bead graphs, creating tasks or epics, understanding status transitions, claiming work via bd ready, or encoding workflow progression. Essential for any agent operating under this workflow.
context: fork
---

# Beads Workflow Encoding

This skill provides the canonical understanding of how workflow progression is encoded in the bead structure. All agents (main, subagents, parallel workers) should internalize these patterns.

Use `bd prime` for quick reference of available beads commands.

## Core Concept: Workflow Lives in Beads

Workflow state is encoded in **epic status**, not file contents. The bead graph is the primary source of context, not file paths.

Key principles:

1. **Status encodes workflow position** — Epic status tells you exactly where work stands
2. **Bead graph is primary context** — Traverse beads before reading files
3. **Artifacts link FROM beads** — Documents are referenced in bead descriptions, not vice versa
4. **Parallel coordination via `bd ready`** — Multiple agents claim independent tasks

## Bead Types

| Type      | When to Create                            | Parent | Status Progression                |
| --------- | ----------------------------------------- | ------ | --------------------------------- |
| `epic`    | Multi-PR features requiring full ceremony | None   | Full 15-state machine             |
| `task`    | Implementation units                      | Epic   | `open` → `in_progress` → `closed` |
| `feature` | Single-PR features                        | None   | Lightweight path                  |
| `bug`     | Bug fixes                                 | None   | Lightweight path                  |

### Research Tasks (Special Pattern)

When `/research` processes a spec with "Research Questions":

1. Create one task bead per question
2. Each task focuses on ONE question deeply
3. Close task when question answered
4. Do NOT attempt all questions in one session

See `references/bead-types-semantic.md` for complete type definitions.

## Status Encoding

### Pattern Recognition

```
needs_X        = Agent should run /<command>
Xing           = Agent actively working
needs_X_review = Agent done, human must approve
```

### Full State Machine (Summary)

```
open → needs_spec → needs_research → researching → needs_research_review
    → needs_design → designing → needs_design_review → needs_plan
    → planning → needs_plan_review → needs_task_breakdown
    → needs_implementation → implementing → needs_implementation_review
    → testing → closed
```

### Command → Status Transitions

| Command                 | On Start       | On Complete                   |
| ----------------------- | -------------- | ----------------------------- |
| `/spec`                 | —              | `needs_research`              |
| `/research`             | `researching`  | `needs_research_review`       |
| `/techdesign`           | `designing`    | `needs_design_review`         |
| `/plan`                 | `planning`     | `needs_plan_review`           |
| `/taskify`              | —              | `needs_implementation`        |
| `/implement`            | `implementing` | `needs_implementation_review` |
| `/implement --validate` | `testing`      | `closed`                      |

### Human Gates

These transitions require human action (not automatic):

- `needs_research_review` → `needs_design`
- `needs_design_review` → `needs_plan`
- `needs_plan_review` → `needs_task_breakdown`
- `needs_implementation_review` → `testing`

See `references/state-machine.md` for complete state definitions and transition rules.

## Context Discovery Protocol

**Do NOT read files directly. Discover context via bead graph.**

1. **Start**: `bd show <task-bead>` — read description, dependencies
2. **Traverse**: Walk upward via parent relationships
3. **Assess**: Upon reaching spec epic, is context sufficient?
   - **YES** → Proceed with implementation
   - **NO** → Continue to step 4
4. **Find prereqs**: Look for research/plan/design child beads of epic
5. **Fallback**: Read documents linked in prereq bead descriptions

### Why Bead-First Discovery?

- **Progressive disclosure**: Load only needed context
- **Self-contained beads**: Agents can work with just descriptions
- **Documents as fallback**: Not primary source
- **Parallel coordination**: Agents find work via `bd ready`

See `references/context-discovery-protocol.md` for detailed algorithm.

## Dependency Patterns

### Parent-Child (Containment)

```bash
bd create "Task: Phase 1" --parent myproject-abc
```

Epic contains tasks. Tasks inherit epic context.

### Blocks (Sequencing)

```bash
bd dep add myproject-xyz myproject-abc  # xyz depends on abc
```

Task xyz cannot start until abc completes. Use for:

- Phase ordering
- Prerequisite tasks
- Convergence points (multiple tasks must complete before next)

See `references/dependency-patterns.md` for sequencing strategies.

## Artifact Linking

Artifacts (spec/research/design/plan) are **documents**, not beads. Link them in epic description:

```markdown
## Documents

- Spec: thoughts/specs/YYYY-MM-DD-feature.md
- Research: thoughts/research/YYYY-MM-DD-feature.md
- Design: thoughts/design/YYYY-MM-DD-feature.md
- Plan: thoughts/plans/YYYY-MM-DD-feature.md
```

This enables context recovery via `bd show <epic>`.

See `references/artifact-linking.md` for linking conventions.

## Commit Tagging

| Commit Type               | Tag With     | Example                                      |
| ------------------------- | ------------ | -------------------------------------------- |
| Spec/research/design/plan | Epic bead ID | `[myproject-abc] spec(feature): Add spec`    |
| Implementation code       | Task bead ID | `[myproject-xyz] feat(feature): Implement X` |

Format: `[<bead-id>] type(scope): message`

See `references/commit-tagging.md` for complete conventions.

## Essential Commands

```bash
bd ready                              # Find unblocked tasks
bd show <id>                          # Read bead + context
bd update <id> --status in_progress   # Claim task
bd close <id>                         # Complete task
bd list --parent <epic-id>            # List epic's children
bd dep add <a> <b>                    # a depends on b
bd blocked                            # Show blocked tasks
bd stats                              # Project health
```

## Lightweight vs Full Ceremony

### Use Full Ceremony When:

- Multi-PR feature
- Touches multiple services
- Architectural decisions needed
- Significant risk or unknowns

### Use Lightweight Path When:

- Single-PR change
- Bug fix
- Obvious implementation

Lightweight: `bd create` → implement → `bd close`

## Bead Description Template

All beads should follow the standard template (see `templates/bead-description.md`):

- **Status**: What's done, in progress, blocked (include phase if from plan)
- **Critical References**: 2-3 key file paths
- **Recent Changes**: Code changes with `file:line` syntax
- **Learnings**: Key discoveries, decisions, what didn't work
- **Artifacts**: Documents produced or updated
- **Blockers**: Current blockers or "None"
- **Next Steps**: Ordered immediate actions
- **Other Notes**: Edge cases, gotchas, tactical info

## Session Handoff

When ending a session with incomplete work:

1. Create handoff as subtask: `bd create "Handoff: <summary>" --parent <work-bead>`
2. Use bead description template
3. Run `bd sync`
4. Provide resume command: `/takeon <handoff-id>`

## Additional Resources

### Reference Files

- **`references/state-machine.md`** — Complete state definitions and transition rules
- **`references/context-discovery-protocol.md`** — Detailed discovery algorithm
- **`references/bead-types-semantic.md`** — When to use each type
- **`references/dependency-patterns.md`** — Sequencing strategies
- **`references/artifact-linking.md`** — Linking documents to epics
- **`references/commit-tagging.md`** — Commit message conventions

### Example Files

- **`examples/epic-full-ceremony.md`** — Epic through all 15 states
- **`examples/task-graph-example.md`** — Dependency-aware breakdown
- **`examples/research-tasks.md`** — Creating from spec questions
