---
description: Create or iterate on a plan doc
model: opus
---

# Implementation Plan

You are tasked with either **creating** new implementation plans or **iterating** on existing ones. You should be skeptical, thorough, and work collaboratively with the user.

## Mode Detection

When this command is invoked, determine the mode:

1. **Create Mode**: No existing plan referenced, or explicit "create" intent
2. **Iterate Mode**: Existing plan file provided, or feedback on existing plan

The process differs based on mode.

---

## ITERATE MODE

When updating an existing plan based on feedback:

### Step 1: Read and Understand Current Plan

1. **Read the existing plan file COMPLETELY** (no limit/offset)
2. **Understand the requested changes**:
   - Parse what the user wants to add/modify/remove
   - Identify if changes require codebase research
   - Determine scope of the update

### Step 2: Research If Needed

**Only spawn research tasks if the changes require new technical understanding.**

If the user's feedback requires understanding new code patterns:

1. Create a research todo list using TodoWrite
2. Spawn parallel sub-tasks (codebase-locator, codebase-analyzer, codebase-pattern-finder)
3. Wait for ALL sub-tasks to complete before proceeding

### Step 3: Present Understanding and Approach

Before making changes, confirm your understanding:

```
Based on your feedback, I understand you want to:
- [Change 1 with specific detail]
- [Change 2 with specific detail]

My research found:
- [Relevant code pattern or constraint]

I plan to update the plan by:
1. [Specific modification to make]
2. [Another modification]

Does this align with your intent?
```

Get user confirmation before proceeding.

### Step 4: Update the Plan

1. **Make focused, precise edits**:
   - Use the Edit tool for surgical changes
   - Maintain the existing structure unless explicitly changing it
   - Keep all file:line references accurate

2. **Ensure consistency**:
   - If adding a new phase, ensure it follows the existing pattern
   - If modifying scope, update "What We're NOT Doing" section
   - If changing approach, update "Implementation Approach" section
   - Maintain the distinction between automated vs manual success criteria

3. **Preserve quality standards**:
   - Include specific file paths and line numbers for new content
   - Write measurable success criteria
   - Keep language clear and actionable

### Step 5: Present Changes

```
I've updated the plan at `thoughts/plans/[filename].md`

Changes made:
- [Specific change 1]
- [Specific change 2]

The updated plan now:
- [Key improvement]

Would you like any further adjustments?
```

### Iterate Mode Guidelines

1. **Be Surgical**: Make precise edits, not wholesale rewrites. Preserve good content that doesn't need changing.
2. **Be Skeptical**: Don't blindly accept change requests that seem problematic. Question vague feedback.
3. **No Open Questions**: If the requested change raises questions, ASK immediately. Do NOT update the plan with unresolved questions.

---

## CREATE MODE

When creating a new implementation plan:

### Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a file path or ticket reference was provided as a parameter, skip the default message
   - Immediately read any provided files FULLY
   - Begin the research process

2. **If no parameters provided**, respond with:

```
I'll help you create a detailed implementation plan. Let me start by understanding what we're building.

Please provide:
1. The task/ticket description (or reference to a ticket file)
2. Any relevant context, constraints, or specific requirements
3. Links to related research, design documents, or previous implementations

I'll analyze this information and work with you to create a comprehensive plan.

Tip: You can also invoke this command with a file directly:
- From spec epic: `/plan bead-abc` (bead ID)
- From design: `/plan thoughts/design/2025-01-08-feature-name.md`
```

Then wait for the user's input.

## Process Steps

### Step 1: Context Gathering & Initial Analysis

1. **Read all mentioned files immediately and FULLY**:
   - **Design documents** (e.g., `thoughts/design/2025-01-08-feature-name.md`) — **preferred input**
   - Ticket files
   - Research documents
   - Related implementation plans
   - **IMPORTANT**: Use the Read tool WITHOUT limit/offset parameters to read entire files
   - **CRITICAL**: DO NOT spawn sub-tasks before reading these files yourself in the main context

2. **If a design document is provided**, extract key planning inputs:
   - **Component responsibilities** → drives phase boundaries
   - **Semantic Test Cases** → drives success criteria
   - **Change Management** → drives deployment phasing
   - **Failure Modes** → drives error handling requirements
   - **Implementation Patterns** → drives code structure

   The design document is your "constitution" — implementation choices should align with its architectural decisions.

3. **Spawn initial research tasks to gather context**:
   Before asking the user any questions, use specialized agents to research in parallel:
   - Use the **codebase-locator** agent to find all files related to the ticket/task
   - Use the **codebase-analyzer** agent to understand how the current implementation works
   - If relevant, use the **thoughts-locator** agent to find any existing thoughts documents about this feature

4. **Read all files identified by research tasks**:
   - After research tasks complete, read ALL files they identified as relevant
   - Read them FULLY into the main context
   - **Skip this step** if a design document was provided (it already contains the necessary context)

5. **Analyze and verify understanding**:
   - Cross-reference the ticket requirements with actual code
   - Identify any discrepancies or misunderstandings
   - Note assumptions that need verification

6. **Present informed understanding and focused questions**:

   ```
   Based on the ticket and my research of the codebase, I understand we need to [accurate summary].

   I've found that:
   - [Current implementation detail with file:line reference]
   - [Relevant pattern or constraint discovered]

   Questions that my research couldn't answer:
   - [Specific technical question that requires human judgment]
   - [Business logic clarification]
   ```

   Only ask questions that you genuinely cannot answer through code investigation.

### Step 2: Research & Discovery

After getting initial clarifications:

1. **If the user corrects any misunderstanding**:
   - DO NOT just accept the correction
   - Spawn new research tasks to verify the correct information
   - Read the specific files/directories they mention

2. **Create a research todo list** using TodoWrite to track exploration tasks

3. **Spawn parallel sub-tasks for comprehensive research**:
   - Use the right agent for each type of research:

   **For deeper investigation:**
   - **codebase-locator** - To find more specific files
   - **codebase-analyzer** - To understand implementation details
   - **codebase-pattern-finder** - To find similar features we can model after

   **For historical context:**
   - **thoughts-locator** - To find any research, plans, or decisions about this area
   - **thoughts-analyzer** - To extract key insights from the most relevant documents

4. **Wait for ALL sub-tasks to complete** before proceeding

5. **Present findings and design options**:

   ```
   Based on my research, here's what I found:

   **Current State:**
   - [Key discovery about existing code]
   - [Pattern or convention to follow]

   **Design Options:**
   1. [Option A] - [pros/cons]
   2. [Option B] - [pros/cons]

   **Open Questions:**
   - [Technical uncertainty]
   - [Design decision needed]

   Which approach aligns best with your vision?
   ```

### Step 3: Plan Structure Development

Once aligned on approach:

1. **Create initial plan outline**:

   ```
   Here's my proposed plan structure:

   ## Overview
   [1-2 sentence summary]

   ## Implementation Phases:
   1. [Phase name] - [what it accomplishes]
   2. [Phase name] - [what it accomplishes]
   3. [Phase name] - [what it accomplishes]

   Does this phasing make sense? Should I adjust the order or granularity?
   ```

2. **Get feedback on structure** before writing details

### Step 4: Detailed Plan Writing

After structure approval:

1. **Write the plan** to `thoughts/plans/YYYY-MM-DD-<bead-id>-description.md`
2. **Use this template structure**:

````markdown
# [Feature/Task Name] Implementation Plan

## Overview

[Brief description of what we're implementing and why]

## Current State Analysis

[What exists now, what's missing, key constraints discovered]

## Desired End State

[Specification of the desired end state and how to verify it]

### Key Discoveries:

- [Important finding with file:line reference]
- [Pattern to follow]
- [Constraint to work within]

## What We're NOT Doing

[Explicitly list out-of-scope items to prevent scope creep]

## Implementation Approach

[High-level strategy and reasoning]

## Phase 1: [Descriptive Name]

### Overview

[What this phase accomplishes]

### Changes Required:

#### 1. [Component/File Group]

**File**: `path/to/file.ext`
**Changes**: [Summary of changes]

```[language]
// Specific code to add/modify
```

### Success Criteria:

#### Automated Verification:

- [ ] Tests pass: `npm test` (or your test command)
- [ ] Type checking passes: `npm run typecheck`
- [ ] Linting passes: `npm run lint`

#### Manual Verification:

- [ ] Feature works as expected when tested via UI
- [ ] Edge case handling verified manually

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation before proceeding to the next phase.

---

## Phase 2: [Descriptive Name]

[Similar structure...]

---

## Testing Strategy

### Unit Tests:

- [What to test]
- [Key edge cases]

### Integration Tests:

- [End-to-end scenarios]

### Manual Testing Steps:

1. [Specific step to verify feature]
2. [Another verification step]

## Performance Considerations

[Any performance implications or optimizations needed]

## Migration Notes

[If applicable, how to handle existing data/systems]

## References

- Design document: `thoughts/design/YYYY-MM-DD-feature-name.md` (if applicable)
- Spec epic: `bd show <bead-id>`
- Related research: `thoughts/research/[relevant].md`
````

### Step 5: Sync and Review

1. **Present the draft plan location**:

   ```
   I've created the initial implementation plan at:
   `thoughts/plans/YYYY-MM-DD-<bead-id>-description.md`

   Please review it and let me know:
   - Are the phases properly scoped?
   - Are the success criteria specific enough?
   - Any technical details that need adjustment?
   ```

2. **Iterate based on feedback** - be ready to:
   - Add missing phases
   - Adjust technical approach
   - Clarify success criteria

3. **Continue refining** until the user is satisfied

## Important Guidelines

1. **Be Skeptical**: Question vague requirements. Don't assume - verify with code.

2. **Be Interactive**: Don't write the full plan in one shot. Get buy-in at each major step.

3. **Be Thorough**: Read all context files COMPLETELY before planning. Research actual code patterns.

4. **Be Practical**: Focus on incremental, testable changes. Include "what we're NOT doing".

5. **Track Progress**: Use TodoWrite to track planning tasks.

6. **No Open Questions in Final Plan**: If you encounter open questions during planning, STOP. Research or ask for clarification immediately. The implementation plan must be complete and actionable.

## Success Criteria Guidelines

**Always separate success criteria into two categories:**

1. **Automated Verification** (can be run by execution agents):
   - Commands that can be run
   - Specific files that should exist
   - Code compilation/type checking
   - Automated test suites

2. **Manual Verification** (requires human testing):
   - UI/UX functionality
   - Performance under real conditions
   - Edge cases that are hard to automate

## Project Patterns

This command should use the **project-patterns skill** (if configured) for project-specific guidance on implementation patterns, testing commands, and conventions.

See `skills/project-patterns/` for the template.

## Epic Integration

### Finding the Epic

When invoked:

1. **If epic bead ID provided**: Use that epic
2. **If design doc provided**: Find epic referencing that design via description search
3. **Check for epics in planning phase**: `bd list --type=epic --status=needs_plan`
4. **Otherwise**: Proceed without epic linkage

### Status Transitions

**On start** (if epic found and status is `needs_plan`):
```bash
bd update <epic-id> --status planning
```

**On completion** (plan document written):
```bash
bd update <epic-id> --status needs_plan_review
```

**Link plan document to epic**: Update the epic's notes with a `## Documents` section.

**Present to user**:
```
Plan complete. Epic <epic-id> advanced to `needs_plan_review`.

Please review the implementation plan. When approved, run:
  bd update <epic-id> --status needs_task_breakdown

Then run `/taskify` to create task beads.
```

## Workflow Position

**Current stage:** `/plan` — "What are the implementation steps?" (procedural)

This is Step 4 of the epic workflow. See the `beads-workflow` skill for the complete state machine.

## Committing Changes

After writing the plan, commit with the **epic bead ID** (if linked):

```bash
git add thoughts/plans/<plan-file>.md
git commit -m "[<epic-id>] Add implementation plan for <feature>"
```

**Do NOT include** auto-generated signatures.
