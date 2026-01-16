---
description: Create or iterate on a technical design doc
model: opus
---

# Technical Design

You are tasked with either **creating** new technical designs or **iterating** on existing ones. Your job is to make architectural decisions: where state lives, how components interact, what can fail, and how to deploy safely.

This command produces self-contained design documents that downstream implementation agents can use autonomously.

## Mode Detection

When this command is invoked, determine the mode:

1. **Create Mode**: No existing design referenced, or explicit "create" intent
2. **Iterate Mode**: Existing design file provided, feedback on existing design, or resolving `[NEEDS CLARIFICATION]` items

---

## ITERATE MODE

When updating an existing design based on feedback or resolving ambiguities:

### When to Use Iterate Mode

- Human feedback changes architectural decisions
- Implementation revealed constraints not considered in original design
- `[NEEDS CLARIFICATION]` sections need resolution
- Tradeoffs need re-evaluation after prototype learnings
- Scope changes require design updates

### Step 1: Read and Understand Current Design

1. **Read the existing design file COMPLETELY** (no limit/offset)
2. **Understand the architectural decisions and their rationale**
3. **Note all `[NEEDS CLARIFICATION]` markers**
4. **Identify downstream artifacts** (plans, implementations)

### Step 2: Check for Downstream Artifacts

Look for plans referencing this design:
```bash
grep -r "[design-filename]" thoughts/plans/
```

Note any implementations that may need updating.

### Step 3: Research If Needed

**Only spawn research tasks if the changes require new technical understanding.**

### Step 4: Present Understanding and Approach

```
Based on your feedback, I understand you want to:
- [Change 1 with specific detail]

Architectural impact:
- [Component X] will now [change in responsibility]
- [Downstream plan Z] may need updating

I plan to update the design by:
1. [Specific modification to make]

Does this align with your intent?
```

### Step 5: Update the Design

1. **Make focused, precise edits**
2. **Preserve design history**: When changing a decision, document what changed and why
3. **Maintain architectural coherence**:
   - If changing state management, update all affected component responsibilities
   - If changing API contracts, update failure modes and test cases
   - Ensure all sections stay aligned

### Step 6: Check Downstream Artifacts

```
This design change may affect:
- Plan: `thoughts/plans/[related-plan].md` - [what needs updating]

Recommended next steps:
1. [Update to plan if needed]
2. [Re-run /implement verify if implementation exists]
```

### Resolving [NEEDS CLARIFICATION] Items

When specifically asked to resolve ambiguities:

1. **Read the clarification context**: What options were presented? What was the recommendation?
2. **Get the resolution from the user** if not specified
3. **Update all occurrences**:
   - Remove inline `[NEEDS CLARIFICATION]` marker
   - Replace with the resolved decision and rationale
   - Move from "Unresolved" to "Resolved" in Open Questions table
4. **Propagate the decision**: Update any sections affected by the resolution

### Iterate Mode Guidelines

1. **Be Surgical**: Make precise edits, not wholesale rewrites
2. **Preserve Rationale**: Never delete design rationale without replacement
3. **Maintain Coherence**: Architectural decisions must be consistent across sections
4. **Track Clarification Resolution**: Ensure no orphaned `[NEEDS CLARIFICATION]` markers remain
5. **Consider Downstream Impact**: Design changes may invalidate existing plans

---

## CREATE MODE

When creating a new technical design:

## When to Use This Command

Use `/techdesign` when:

- Adding significant new functionality that touches multiple components
- The "right" approach isn't obvious and requires tradeoff analysis
- You need to design new APIs, state management, or component interactions
- There are deployment ordering or migration concerns

Skip this command for:

- Simple bug fixes or single-file changes
- Features where the design is already specified
- Pure refactoring with no behavioral changes

## Initial Response

When this command is invoked:

1. **If context was provided** (ticket, research doc, requirements file):
   - Read it FULLY first (no limit/offset)
   - Check `thoughts/research/` for relevant prior research
   - Begin the design process

2. **If no context provided**, respond with:

   ```
   I'll help you design how to extend the system for a new feature.

   Please provide:
   1. The feature requirements (ticket path, description, or external spec link)
   2. Any relevant research documents (or I can run /research first)
   3. Key constraints or non-negotiables you already know about

   Tip: For complex features, run `/research` first to document the current system.
   ```

   Then wait for the user's input.

## Process Steps

### Step 1: Understand Requirements

1. **Read all provided context COMPLETELY** (no limit/offset)
2. **Identify what needs to be designed**:
   - New state that must be persisted
   - New APIs (endpoints, events, internal messages)
   - New component interactions or data flows
   - Changes to existing behavior

3. **Present initial understanding**:

   ```
   Based on the requirements, I understand we need to design:
   - [New capability 1]
   - [New capability 2]

   This will likely touch:
   - [Component 1] - for [reason]
   - [Component 2] - for [reason]
   ```

### Step 2: Research Current Architecture

1. **Spawn parallel research tasks**:
   - **codebase-locator** - Find files related to components you'll extend
   - **codebase-analyzer** - Understand how existing similar features work
   - **codebase-pattern-finder** - Find patterns to follow
   - **thoughts-locator** - Find prior design decisions or research

2. **Wait for ALL research tasks to complete**

3. **Read key files FULLY** into main context

4. **Synthesize architectural constraints**:
   - What patterns must be followed?
   - What constraints does the existing architecture impose?

### Step 3: Design the Extension

Apply architectural reasoning to each decision area.

> **NOTE**: Consult your project's `project-patterns` skill for project-specific heuristics on state management, type selection, testing patterns, and critical pitfalls.

#### State Management

For each piece of state, explicitly decide:

- **Where it lives**: Database, cache, memory, external service
- **Why it lives there**: Persistence needs, access patterns, consistency requirements
- **How it's accessed**: Direct, through repository, via API

#### Type/Precision Decisions

For EVERY new data field, explicitly decide on the appropriate type based on:

- Precision requirements (monetary values, percentages, etc.)
- Overflow concerns
- Storage and performance tradeoffs

#### Component Responsibilities

For each component touched, specify:

- **Owns**: What state/behavior it's responsible for
- **Publishes**: What events/messages it sends
- **Subscribes to**: What events/messages it receives

#### API Contracts

For each new API, specify:

- Route and method
- Request/response structure
- Validation rules (stateless vs stateful)
- Error responses

#### Failure Modes

Cover at minimum:

- Validation failures
- Downstream unavailability
- Race conditions
- State inconsistency

#### Semantic Test Cases

For EVERY component in scope, describe expected behavior in Given-When-Then format:

| Scenario | Given | When | Then |
|----------|-------|------|------|
| Happy path | [preconditions] | [action] | [expected outcome] |
| Validation failure | [preconditions] | [action] | [expected error] |
| Edge case | [preconditions] | [action] | [expected handling] |

### Step 4: Change Management Analysis

For safe deployment, analyze:

#### Feature Flags

- What flags are needed?
- Can they be safely toggled?

#### Migration Requirements

| Type | Required? | Details |
|------|-----------|---------|
| Database schema | ? | [fields added] |
| Data backfill | ? | [historical data handling] |

#### Deployment Order

1. [First step]
2. [Second step]
3. ...

#### Rollback Strategy

- What happens if we need to revert?
- Is the change additive (safe) or destructive?

### Step 5: Identify and Mark Ambiguities

Before presenting the design, actively scan for unresolved decisions:

1. **For each component**: Is the responsibility clear? If not, mark `[NEEDS CLARIFICATION]`
2. **For each state decision**: Is the location justified? If assumptions exist, mark them
3. **For each API**: Are the error cases defined? If not, flag them
4. **For each failure mode**: Is the mitigation actionable? If vague, flag it

### Step 6: Present Design for Review

Before writing the document, present a summary:

```
Here's my proposed design for [Feature]:

**Key Decisions:**
1. State lives in [location] because [reason]
2. [Component] owns [responsibility] because [reason]
3. New API: [endpoint] with [key characteristics]

**State Management:**
- [Location A]: [what state]
- [Location B]: [what state]

**Ambiguities Flagged [NEEDS CLARIFICATION]:**
| Location | Question | Recommendation |
|----------|----------|----------------|
| [area] | [question] | [recommendation] |

Does this direction make sense? Please resolve any flagged ambiguities before I finalize.
```

**CRITICAL**: Do NOT finalize the design with unresolved `[NEEDS CLARIFICATION]` items.

### Step 7: Write Design Document

After approval, write to `thoughts/design/YYYY-MM-DD-<description>.md`:

```markdown
---
date: [ISO timestamp]
author: [Author name]
git_commit: [Current commit hash]
branch: [Current branch]
status: draft | approved
---

# Design: [Feature Name]

## Overview

[1-2 paragraph summary of what we're designing and why]

## Requirements

[Key requirements this design must satisfy]

## Design Principles

[Guiding principles for this design]

## System Components Touched

| Component | Responsibility | Changes Required |
|-----------|----------------|------------------|
| [Component] | [What it does] | [What changes] |

## State Management

### Design Rationale

[Why state is split this way]

### [Location A] State

[What's stored here and why]

### [Location B] State

[What's stored here and why]

## Type Decisions

| Field | Type | Rationale |
|-------|------|-----------|
| [field] | [type] | [why] |

## API Contracts

### [METHOD] /[route]

**Request:**
```json
{
  "field": "value"
}
```

**Response:**
```json
{
  "result": { ... }
}
```

**Validation:**
1. [stateless validations]
2. [stateful validations]

## Interaction Patterns

[Message flow or sequence diagram]

## Failure Modes

| Failure | Detection | Impact | Mitigation |
|---------|-----------|--------|------------|
| [failure] | [how] | [what breaks] | [how handled] |

## Change Management

### Feature Flags

| Flag | Purpose | Reversible? |
|------|---------|-------------|
| [name] | [purpose] | [Yes/No] |

### Migration Requirements

| Type | Required? | Details |
|------|-----------|---------|
| Database schema | [Yes/No] | [description] |
| Data backfill | [Yes/No] | [description] |

### Deployment Order

1. [Step]
2. [Step]

### Rollback Strategy

[What happens if issues detected]

## Semantic Test Cases

### [Component 1]: [Feature]

| Scenario | Given | When | Then |
|----------|-------|------|------|
| [name] | [preconditions] | [action] | [outcome] |

## Implementation Patterns

[Relevant patterns applied to this feature. Consult project-patterns skill for guidance.]

## Alternatives Considered

| Alternative | Why Rejected |
|-------------|--------------|
| [option] | [reason] |

## Open Questions

| Question | Options | Recommendation | Status |
|----------|---------|----------------|--------|
| [question] | A, B, C | [recommendation] | [NEEDS CLARIFICATION] |

## References

- Research: `thoughts/research/[relevant].md`
- Similar implementation: `[file:line]`
```

### Step 8: Handoff to Planning

After the design is approved:

```
The design document is complete at:
`thoughts/design/YYYY-MM-DD-<description>.md`

Next step: Run `/plan thoughts/design/YYYY-MM-DD-<description>.md` to create a phased implementation plan.
```

---

## Project Patterns

This command should use the **project-patterns skill** (if configured) for project-specific guidance on:

- State management heuristics
- Type/precision decisions
- Component patterns
- Testing requirements
- Critical pitfalls

See `skills/project-patterns/` for the template. Projects should customize this skill with their own tribal knowledge.

---

## Important Guidelines

1. **Apply Heuristics Explicitly**: State management, type decisions, and testing must be explicit.

2. **Be Evaluative**: Compare tradeoffs explicitly. Make recommendations with reasoning.

3. **Be Skeptical**: Question requirements. Identify hidden complexity.

4. **Be Interactive**: Get buy-in on major decisions. Resolve open questions before finalizing.

5. **Produce Self-Contained Output**: The design document must carry forward all patterns and constraints.

6. **Be Obsessive About Ambiguity**: Mark EVERY unresolved decision with `[NEEDS CLARIFICATION]`.

## Epic Integration

### Finding the Epic

When invoked:

1. **If epic bead ID provided**:
   - Spawn `bd-discoverer` agent to gather context from the epic
   - The agent will return spec, research links, and key findings
   - Use that epic for status tracking
2. **If design context provided**: Check for epics in design phase: `bd list --type=epic --status=needs_design`
3. **Otherwise**: Proceed without epic linkage

### Status Transitions

**On start** (if epic found and status is `needs_design`):
```bash
bd update <epic-id> --status designing
```

**On completion** (design document written):
```bash
bd update <epic-id> --status needs_design_review
```

**Present to user**:
```
Design complete. Epic <epic-id> advanced to `needs_design_review`.

Please review the design document. When approved, run:
  bd update <epic-id> --status needs_plan

Then run `/plan` to proceed.
```

## Workflow Position

**Current stage:** `/techdesign` — "How should we extend it?" (evaluative)

This is Step 3 of the epic workflow. See the `beads-workflow` skill for the complete state machine.

## Committing Changes

After writing the design document, commit with the **epic bead ID** (if linked):

```bash
git add thoughts/design/<design-file>.md
git commit -m "[<epic-id>] Add technical design for <feature>"
```

**Do NOT include** auto-generated signatures.
