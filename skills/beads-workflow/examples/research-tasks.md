# Example: Creating Research Tasks from Spec

This example shows how to create focused research task beads from a spec with research questions.

## Scenario

Spec epic `myproject-abc` has the following research questions in `thoughts/specs/2025-01-08-feature.md`:

```markdown
## Research Questions

1. How is user authentication currently implemented?
2. What validation occurs in the API handlers before processing requests?
3. How do existing tests verify authentication behavior?
4. What data models are used for user sessions?
5. Are there any existing helper functions for token validation?
```

## Step 1: Check for Existing Research Tasks

```bash
$ bd list --parent myproject-abc

myproject-abc (epic): Feature name
  Status: needs_research
  Children: (none)
```

No research tasks exist yet.

## Step 2: Create Research Task Beads

Create ONE task per research question:

```bash
# Question 1
$ bd create "Research: Auth implementation" \
  --type task \
  --parent myproject-abc \
  --priority 2 \
  --description "$(cat <<'EOF'
## Research Question
How is user authentication currently implemented?

## Scope
- Find auth entry points in the codebase
- Document the authentication flow
- Identify relevant state and data structures
- Note any configuration affecting auth

## Deliverable
Update thoughts/research/2025-01-08-user-auth.md with findings.
EOF
)"
# Created: myproject-abc.1

# Question 2
$ bd create "Research: API validation" \
  --type task \
  --parent myproject-abc \
  --priority 2 \
  --description "$(cat <<'EOF'
## Research Question
What validation occurs in the API handlers before processing requests?

## Scope
- Find request validation in API handlers
- Document stateless vs stateful validation split
- Identify auth-related checks (if any)

## Deliverable
Update thoughts/research/2025-01-08-user-auth.md with findings.
EOF
)"
# Created: myproject-abc.2

# Question 3
$ bd create "Research: Auth tests" \
  --type task \
  --parent myproject-abc \
  --priority 2 \
  --description "$(cat <<'EOF'
## Research Question
How do existing tests verify authentication behavior?

## Scope
- Find auth-related test files
- Document test patterns used
- Identify gaps in coverage

## Deliverable
Update thoughts/research/2025-01-08-user-auth.md with findings.
EOF
)"
# Created: myproject-abc.3

# Question 4
$ bd create "Research: Session data models" \
  --type task \
  --parent myproject-abc \
  --priority 2 \
  --description "$(cat <<'EOF'
## Research Question
What data models are used for user sessions?

## Scope
- Find session-related models/types
- Document data structures
- Note any naming conventions

## Deliverable
Update thoughts/research/2025-01-08-user-auth.md with findings.
EOF
)"
# Created: myproject-abc.4

# Question 5
$ bd create "Research: Token helpers" \
  --type task \
  --parent myproject-abc \
  --priority 2 \
  --description "$(cat <<'EOF'
## Research Question
Are there any existing helper functions for token validation?

## Scope
- Search for token validation utilities
- Document precision/security handling
- Note any constants or configuration

## Deliverable
Update thoughts/research/2025-01-08-user-auth.md with findings.
EOF
)"
# Created: myproject-abc.5
```

## Step 3: Verify Research Tasks

```bash
$ bd list --parent myproject-abc

myproject-abc.1: Research: Auth implementation [open]
myproject-abc.2: Research: API validation [open]
myproject-abc.3: Research: Auth tests [open]
myproject-abc.4: Research: Session data models [open]
myproject-abc.5: Research: Token helpers [open]

$ bd ready
myproject-abc.1: Research: Auth implementation [open]
myproject-abc.2: Research: API validation [open]
myproject-abc.3: Research: Auth tests [open]
myproject-abc.4: Research: Session data models [open]
myproject-abc.5: Research: Token helpers [open]
```

All 5 research tasks are ready (no dependencies between them).

## Step 4: Work on ONE Task

**Important:** Do NOT attempt all questions in one session.

```bash
# Claim one task
$ bd update myproject-abc.1 --status in_progress

# Focus research on just this question
# ... deep research on auth implementation ...

# When complete, update description with learnings
$ bd update myproject-abc.1 --description "$(cat <<'EOF'
## Research Question
How is user authentication currently implemented?

## Findings

### Entry Points
- Login handler: `src/api/auth/login.ts:42`
- Token validation: `src/middleware/auth.ts:156`

### Authentication Flow
1. Request received at login endpoint
2. Credentials validated against database
3. JWT token generated with user claims
4. Token returned in response

### Key Structures
- `User` model at `src/models/user.ts:28`
- Uses `TOKEN_EXPIRY_SECONDS` constant (3600)

### Configuration
- `AUTH_SECRET` environment variable required
- Token expiry configurable via env

## Learnings
- Auth is stateless (JWT-based)
- Existing pattern at src/middleware/auth.ts shows correct flow

## Deliverable
Updated thoughts/research/2025-01-08-user-auth.md
EOF
)"

# Close the task
$ bd close myproject-abc.1 --reason "Research complete, findings documented"
```

## Step 5: Present Progress

```bash
$ bd list --parent myproject-abc

myproject-abc.1: Research: Auth implementation [closed]
myproject-abc.2: Research: API validation [open]
myproject-abc.3: Research: Auth tests [open]
myproject-abc.4: Research: Session data models [open]
myproject-abc.5: Research: Token helpers [open]
```

Inform user:

```
Research task myproject-abc.1 complete: "Auth implementation"

Remaining research tasks for this epic:
- myproject-abc.2: API validation (open)
- myproject-abc.3: Auth tests (open)
- myproject-abc.4: Session data models (open)
- myproject-abc.5: Token helpers (open)

Run `/research myproject-abc` again to tackle the next question.
```

## Key Principles

### Why One Question Per Task?

1. **Depth over breadth**: Deep research on ONE question produces better docs
2. **Session boundaries**: Natural stopping point at task completion
3. **Progress visibility**: Clear tracking of which questions are answered
4. **Prioritization**: User can choose which questions matter most
5. **Parallel execution**: Multiple agents can research different questions

### Why Update Bead Description?

The bead description becomes the research summary:
- Future sessions can read it without loading documents
- Learnings are preserved in the bead graph
- Supports the Context Discovery Protocol

### Research Document vs Bead Description

- **Research document** (`thoughts/research/...`): Detailed findings with code references
- **Bead description**: Summary of key learnings and references

Both are updated when closing a research task.

## Anti-Patterns

### Don't: Attempt All Questions at Once

```bash
# WRONG: Claim all tasks
bd update myproject-abc.1 --status in_progress
bd update myproject-abc.2 --status in_progress
bd update myproject-abc.3 --status in_progress
bd update myproject-abc.4 --status in_progress
bd update myproject-abc.5 --status in_progress
# ... shallow research on all ...
```

### Don't: Skip Bead Description Updates

```bash
# WRONG: Close without updating description
bd close myproject-abc.1
# Learnings lost!
```

### Don't: Create Single Mega-Task

```bash
# WRONG: One task for all questions
bd create "Research: All auth questions" --parent myproject-abc
# ... too broad, unclear progress ...
```
