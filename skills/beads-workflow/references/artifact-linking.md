# Artifact Linking

This document describes how to link artifacts (documents) to beads.

## Core Principle

**Artifacts are documents. Beads are work items. Link artifacts FROM bead descriptions.**

Artifacts live in `thoughts/` directories:
- `thoughts/specs/` — Specifications
- `thoughts/research/` — Codebase research
- `thoughts/design/` — Technical designs
- `thoughts/plans/` — Implementation plans

Beads live in `.beads/issues.jsonl` and reference artifacts via their descriptions.

## Linking Convention

### Epic Description Format

When artifacts are created, update the epic description with a `## Documents` section:

```markdown
## Documents
- Spec: thoughts/specs/YYYY-MM-DD-feature-name.md
- Research: thoughts/research/YYYY-MM-DD-feature-name.md
- Design: thoughts/design/YYYY-MM-DD-feature-name.md
- Plan: thoughts/plans/YYYY-MM-DD-feature-name.md
```

### Updating Epic Notes

```bash
bd update <epic-id> --notes "$(cat <<'EOF'
## Documents
- Spec: thoughts/specs/2025-01-08-margin-validation.md
- Research: thoughts/research/2025-01-08-margin-validation.md
- Design: thoughts/design/2025-01-08-margin-validation.md
- Plan: thoughts/plans/2025-01-08-margin-validation.md
EOF
)"
```

### Progressive Linking

As each artifact is created, append to the Documents section:

```bash
# After /spec
bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-feature.md"

# After /research
bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-feature.md
- Research: thoughts/research/2025-01-08-feature.md"

# After /techdesign
bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-feature.md
- Research: thoughts/research/2025-01-08-feature.md
- Design: thoughts/design/2025-01-08-feature.md"

# After /plan
bd update myproject-abc --notes "## Documents
- Spec: thoughts/specs/2025-01-08-feature.md
- Research: thoughts/research/2025-01-08-feature.md
- Design: thoughts/design/2025-01-08-feature.md
- Plan: thoughts/plans/2025-01-08-feature.md"
```

## Naming Conventions

### File Naming

Format: `YYYY-MM-DD-<description>.md`

- Use today's date when creating
- Use kebab-case for description
- Keep description brief but descriptive

Examples:
- `2025-01-08-margin-validation.md`
- `2025-01-08-non-liq-subaccount-capabilities.md`
- `2025-01-08-myproject-abc-feature-name.md` (with bead ID)

### With Bead ID

For plans, include the bead ID in the filename:

Format: `YYYY-MM-DD-<bead-id>-<description>.md`

Example: `2025-01-08-myproject-abc-margin-validation.md`

This creates explicit traceability between plan and epic.

## Multiple Research Documents

When multiple research sessions occur, list them all:

```markdown
## Documents
- Spec: thoughts/specs/2025-01-08-feature.md
- Research:
  - thoughts/research/2025-01-08-margin-check-locations.md
  - thoughts/research/2025-01-09-validation-ordering.md
  - thoughts/research/2025-01-10-error-propagation.md
- Design: thoughts/design/2025-01-11-feature.md
- Plan: thoughts/plans/2025-01-12-myproject-abc-feature.md
```

## Context Recovery

When an agent needs to recover context:

1. **Read epic description:**
   ```bash
   bd show myproject-abc
   ```

2. **Extract document paths from `## Documents` section**

3. **Read relevant documents:**
   ```bash
   cat thoughts/design/2025-01-08-feature.md
   cat thoughts/plans/2025-01-08-myproject-abc-feature.md
   ```

This enables full context recovery from just the epic bead ID.

## Task-Level Artifacts

Tasks typically don't create new artifacts, but may reference them:

```markdown
## Critical References
- Plan: thoughts/plans/2025-01-08-myproject-abc-feature.md (Phase 2)
- Pattern: src/services/auth/validation.ts:42-98

## Artifacts Updated
- thoughts/plans/2025-01-08-myproject-abc-feature.md (marked Phase 2 complete)
```

## Best Practices

### Do:
- Link artifacts immediately after creation
- Use consistent naming conventions
- Include all artifacts in the Documents section
- Update plan checkboxes as implementation progresses

### Don't:
- Store artifact content in bead descriptions (link instead)
- Create beads for artifacts (they're documents, not work items)
- Forget to update links when artifacts are renamed
- Duplicate artifact content across beads

## Relationship to Context Discovery

The artifact linking pattern supports the Context Discovery Protocol:

1. Agent starts with task bead
2. Traverses to epic via parent
3. Reads `## Documents` section from epic description
4. Reads linked artifacts as needed

This creates a clean separation:
- **Beads** = Work coordination (status, dependencies, assignments)
- **Artifacts** = Knowledge artifacts (spec, research, design, plan)
- **Links** = Connection between them (in bead descriptions)
