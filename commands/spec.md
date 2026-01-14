---
description: Create or iterate on a spec
model: opus
---

# Spec

You are tasked with either **creating** a new specification or **iterating** on an existing one. Your mission is to help externalize everything known about a problem and the solution needed until it's actionable.

This command handles both modes - creating from scratch or progressively clarifying an existing spec.

## Initial Response -- Resolve Spec Filepath

1. **If an epic bead ID is provided** (e.g., `/spec myproject-abc`):
   - Run `bd show <bead-id>` to read the epic
   - Check epic description for existing spec link
   - If no spec exists, create one at `thoughts/specs/YYYY-MM-DD-<title-slug>.md`
   - Read the spec file and proceed to assessment

2. **If a spec file path is provided** (e.g., `/spec thoughts/specs/feature.md`):
   - Read the spec file FULLY
   - Search for epic referencing this spec: `bd list --type=epic`
   - Proceed to assessment

3. **If nothing provided**:
   - Check for in-progress epics: `bd list --type=epic --status=needs_spec`
   - If exactly one found, offer to use it
   - Otherwise, ask for epic ID or spec file path

## Spec Template

```markdown
# Spec: [Feature Name]

## Problem

[What's broken? Who's affected? What's the impact?]

## Desired Outcome

[What should be true when done? WHAT, not HOW]

## Definition of Done

"When this is done, [who?] will (not) be able to [what?]"

## Success Metrics

- [ ] [Quantifiable metric 1]
- [ ] [Quantifiable metric 2]

## Non-Goals

- NOT doing: [explicit exclusion 1]
- NOT doing: [explicit exclusion 2]

## Constraints

- Technical: [constraints]
- Security: [requirements]
- Performance: [budgets]

## Risks

| Risk   | Likelihood | Impact | Mitigation   |
| ------ | ---------- | ------ | ------------ |
| [risk] | H/M/L      | H/M/L  | [mitigation] |

## Research Questions

- [ ] [Unknown to resolve before design]
- [ ] [Another unknown]
```

## Process Steps

### Step 1: Assess Current State

For each section of the spec template, evaluate:

| Section | Status | Notes |
|---------|--------|-------|
| Problem | [Complete/Partial/Missing] | [what's missing] |
| Desired Outcome | [Complete/Partial/Missing] | [what's missing] |
| Definition of Done | [Complete/Partial/Missing] | [what's missing] |
| Success Metrics | [Complete/Partial/Missing] | [what's missing] |
| Non-Goals | [Complete/Partial/Missing] | [what's missing] |
| Constraints | [Complete/Partial/Missing] | [what's missing] |
| Risks | [Complete/Partial/Missing] | [what's missing] |
| Research Questions | [Complete/Partial/Missing] | [what's missing] |

### Step 2: Targeted Questions

Ask focused questions to fill gaps:

```
Based on my analysis, the spec needs clarification in these areas:

1. **[Section]**: [Specific question]
2. **[Section]**: [Specific question]
3. **[Section]**: [Specific question]
```

2. Pick ONE focused question to ask.

```
Let's start with [most critical gap]. [Question]
```

3. Update spec file with response.
4. Repeat Steps 2 and 3 until specification is watertight.
5. Track complete spec in epic bead.

```bash
bd update <epic-id> --status needs_research

bd update <epic-id> --notes "$(cat <<'EOF'
## Documents
- Spec: thoughts/specs/YYYY-MM-DD-feature.md
EOF
)"
```

6. Summarize.

```
Spec complete. Epic <epic-id> advanced to `needs_research`.

**Problem**: [one-liner]
**Desired Outcome**: [one-liner]
**Definition of Done**: [statement]

Key research questions to resolve:
1. [question]
2. [question]

Next step: `/research` to investigate research questions.
```

## Spec Readiness Criteria

A spec is ready for research when:

- [ ] Problem is clearly stated with impact
- [ ] Desired outcome describes end state (not implementation)
- [ ] Definition of Done is testable
- [ ] At least one success metric is quantifiable
- [ ] Non-goals explicitly exclude adjacent scope
- [ ] Critical constraints are identified
- [ ] Top 3 risks are documented with mitigations
- [ ] Research questions are specific and answerable

## Important Guidelines

1. **Be Skeptical**: Challenge vague statements. "Make it faster" is not a spec.
2. **Separate What from How**: Specs describe desired outcomes, not solutions.
3. **Quantify Where Possible**: "Improve performance" → "Reduce latency to <100ms p99"
4. **Explicit Non-Goals**: Prevent scope creep by stating what's NOT included.
5. **Testable DoD**: If you can't verify it, it's not done.

## Workflow Position

**Current stage:** `/spec` — "What are we building?" (specification)

This is Step 1 of the epic workflow. See the `beads-workflow` skill for the complete state machine.

## Committing Changes

After writing or updating the spec document, commit with the **epic bead ID**:

```bash
git add thoughts/specs/<spec-file>.md
git commit -m "[<epic-id>] Add spec: <feature name>"
```

**Do NOT include** auto-generated signatures.
