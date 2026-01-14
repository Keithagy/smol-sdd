# Bead Description Template

Use this template for rich bead descriptions that support context discovery and session handoff.

## Template

```markdown
## Status
[What's done, in progress, blocked. Include phase reference if from plan]

## Critical References
- [file:line - brief description]
- [file:line - brief description]

## Recent Changes
- [file:line - what was changed]
- [file:line - what was changed]

## Learnings
- [Key discovery 1]
- [Decision made and why]
- [What didn't work and why]

## Artifacts
- [Document path - description]

## Blockers
[Current blockers or "None"]

## Next Steps
1. [Immediate action 1]
2. [Immediate action 2]
3. [Next priority]

## Other Notes
[Edge cases, gotchas, tactical info]
```

## Usage

### For Task Beads

Focus on:
- Specific implementation status
- Code references with line numbers
- Learnings that affect subsequent tasks
- Clear next steps

### For Handoff Beads

Focus on:
- Complete status picture
- ALL critical file references
- Comprehensive learnings
- Unambiguous next steps

### Example

```markdown
## Status
Phase 2 of 3 complete. Token validation implemented, tests passing.

## Critical References
- src/middleware/auth.ts:42 - main validation entry point
- src/services/session.ts:156 - session storage pattern

## Recent Changes
- src/middleware/auth.ts:42-89 - added JWT validation
- src/services/session.ts:200-215 - added refresh token handling

## Learnings
- Token refresh must happen BEFORE validation to avoid race
- Session store uses Redis with 1hr TTL
- Existing test pattern at auth.test.ts:34 works well

## Artifacts
- thoughts/research/2025-01-15-auth-implementation.md

## Blockers
None

## Next Steps
1. Add rate limiting middleware
2. Update API docs
3. Run full integration test suite

## Other Notes
- Watch out for clock skew in token expiry
- Redis connection pool is limited to 10 connections
```
