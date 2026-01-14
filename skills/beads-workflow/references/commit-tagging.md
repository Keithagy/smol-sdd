# Commit Tagging Conventions

This document describes how to tag git commits with bead IDs for traceability.

## Format

```
[<bead-id>] <type>(<scope>): <message>
```

Components:
- `<bead-id>` — The bead this commit relates to
- `<type>` — Conventional commit type
- `<scope>` — Feature or area name
- `<message>` — Concise description of the change

## Which Bead ID to Use

| Commit Type | Tag With | Example |
|-------------|----------|---------|
| Spec document | Epic bead ID | `[myproject-abc] spec(auth): Add specification` |
| Research document | Epic bead ID | `[myproject-abc] research(auth): Document auth flow locations` |
| Design document | Epic bead ID | `[myproject-abc] design(auth): Add technical design` |
| Plan document | Epic bead ID | `[myproject-abc] plan(auth): Add implementation plan` |
| Implementation code | Task bead ID | `[myproject-xyz] feat(auth): Add validation logic` |
| Bug fix | Bug bead ID | `[myproject-bug1] fix(auth): Correct token expiry handling` |
| Test code | Task bead ID | `[myproject-xyz] test(auth): Add integration tests` |
| Refactoring | Task bead ID | `[myproject-xyz] refactor(auth): Extract validation helper` |

## Conventional Commit Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes nor adds |
| `test` | Adding or updating tests |
| `docs` | Documentation only |
| `chore` | Maintenance tasks |
| `spec` | Specification document |
| `research` | Research document |
| `design` | Design document |
| `plan` | Implementation plan |

## Examples

### Artifact Commits (Epic ID)

```bash
# Spec
git commit -m "[myproject-abc] spec(user-auth): Add specification for user authentication"

# Research
git commit -m "[myproject-abc] research(user-auth): Document existing auth implementation"

# Design
git commit -m "[myproject-abc] design(user-auth): Add technical design for API integration"

# Plan
git commit -m "[myproject-abc] plan(user-auth): Add phased implementation plan"
```

### Implementation Commits (Task ID)

```bash
# Feature code
git commit -m "[myproject-xyz] feat(auth): Add token validation middleware"

# Tests
git commit -m "[myproject-xyz] test(auth): Add unit tests for validation logic"

# Refactoring
git commit -m "[myproject-xyz] refactor(auth): Extract auth helpers to separate module"

# Bug fix
git commit -m "[myproject-xyz] fix(auth): Handle expired tokens correctly"
```

### Multi-Line Commit Messages

For complex changes, use multi-line format:

```bash
git commit -m "$(cat <<'EOF'
[myproject-xyz] feat(auth): Add multi-factor authentication

- Implement TOTP validation
- Add backup code support
- Handle edge case for clock skew

Refs: Phase 2 of thoughts/plans/2025-01-08-user-auth.md
EOF
)"
```

## What NOT to Include

**Never include:**
- Auto-generated signatures
- `🤖 Generated with [Claude Code](https://claude.com/claude-code)`
- `Co-Authored-By: Claude ...`
- Model names or AI attribution

**These are explicitly prohibited** per project conventions.

## Automated Commits (No Tag)

Some commits don't need bead tags:

```bash
# Beads sync (automated by daemon)
chore(beads): bd sync 2025-01-08 14:30:00

# Generated code (automated by codegen)
chore(codegen): Regenerate API types

# Dependency updates
chore(deps): Update dependencies
```

## Commit Workflow

### For Artifact Work

```bash
# 1. Create/update artifact
# 2. Stage the file
git add thoughts/specs/2025-01-08-feature.md

# 3. Commit with epic ID
git commit -m "[myproject-abc] spec(feature-name): Add specification"
```

### For Implementation Work

```bash
# 1. Make code changes
# 2. Stage the files
git add src/services/auth/validation.ts
git add src/services/auth/validation.test.ts

# 3. Commit with task ID
git commit -m "[myproject-xyz] feat(auth): Add validation logic"
```

## Separating Artifact and Code Commits

**Important:** Commit artifacts separately from code changes.

```bash
# First commit: artifacts (using epic ID)
git add thoughts/design/2025-01-08-feature.md
git commit -m "[myproject-abc] design(feature): Add technical design"

# Second commit: implementation (using task ID)
git add src/services/...
git commit -m "[myproject-xyz] feat(feature): Implement Phase 1"
```

This separation:
- Keeps artifact history clean
- Makes code review easier
- Allows independent rollback

## Traceability

The bead ID in commits enables:

```bash
# Find all commits for an epic
git log --oneline --grep="\[myproject-abc\]"

# Find all commits for a task
git log --oneline --grep="\[myproject-xyz\]"

# Find commits for a specific phase
git log --oneline --grep="\[myproject-xyz\]" -- src/services/auth/
```

## Integration with PR Workflow

When creating PRs:
1. Commits should already have bead tags
2. PR description references the epic/task
3. After merge, update bead status

```bash
# After PR merge
bd close myproject-xyz --reason "Merged in PR #123"
```
