---
name: project-patterns
description: Project-specific architectural patterns and tribal knowledge. Copy this skill to your project's .claude/skills/ and customize with your team's conventions, decision trees, and critical pitfalls.
---

# Project Patterns (Template)

> **This is a template skill.** Copy it to your project's `.claude/skills/project-patterns/` directory and customize it with your team's tribal knowledge.

This skill provides authoritative guidance on architectural patterns and implementation practices specific to YOUR project. Claude automatically invokes this skill when working on design, planning, or implementation tasks.

## How to Use This Template

1. Copy this entire `project-patterns/` directory to your project's `.claude/skills/`
2. Edit this `SKILL.md` to describe your project's architecture
3. Add your decision trees and patterns to `patterns/`
4. Document critical pitfalls in `examples/pitfalls.md`
5. Commands like `/plan` and `/techdesign` will automatically reference this skill

## When to Use This Skill

Configure this table for YOUR project:

| Pattern Area | Use When |
|-------------|----------|
| State Management | _Deciding where state lives (e.g., database vs cache vs memory)_ |
| Type Selection | _Choosing data types for precision-critical values_ |
| Message Schemas | _Adding/modifying API schemas or message types_ |
| Service Patterns | _Implementing service layers, handlers, middleware_ |
| Testing Patterns | _Writing tests for your specific tech stack_ |
| Critical Pitfalls | _Avoiding mistakes that corrupt state or cause outages_ |

## Quick Decision Trees

Create decision trees for common architectural decisions in your project:

### Where Should State Live?

```
_Customize this for your architecture_

Is it user-facing data?
├─ Yes → Primary database
└─ No
   ├─ Is it frequently accessed? → Cache layer
   └─ Is it temporary/derived? → In-memory
```

### What Data Type?

```
_Customize this for your language/framework_

Is it monetary or precision-critical?
├─ Yes → [Your precision type, e.g., Decimal, BigInt]
└─ No
   ├─ Is it an identifier? → [Your ID type]
   └─ Is it a counter? → [Your counter type]
```

## Pattern Files

Create detailed patterns in the `patterns/` directory:

| Pattern | File | Phase |
|---------|------|-------|
| State Management | [patterns/state-management.md](patterns/state-management.md) | Design |
| Type Selection | [patterns/type-selection.md](patterns/type-selection.md) | Design |
| Service Patterns | [patterns/service-patterns.md](patterns/service-patterns.md) | Implementation |
| Testing Patterns | [patterns/testing-patterns.md](patterns/testing-patterns.md) | Implementation |
| Critical Pitfalls | [examples/pitfalls.md](examples/pitfalls.md) | Both |

## Phase Appropriateness

**Design Phase** (what to build):
- State management decisions
- Type/precision choices
- Component responsibilities
- API contract shapes
- Failure mode analysis

**Implementation Phase** (how to build):
- Schema/codegen commands
- File path conventions
- Service/handler patterns
- Testing commands

## Service Topology

_Document your project's services here:_

```
[Service A] → [Service B] → [Service C]
     ↓
[Database]
```

## Core Libraries

_Document your project's key libraries and utilities:_

- **Types:** `path/to/types/`
- **Utilities:** `path/to/utils/`
- **Validation:** `path/to/validation/`

## Common File Patterns

_Document where to find common file types:_

- Service entry points: `src/services/*/index.ts`
- API handlers: `src/api/*Handler.ts`
- Database models: `src/models/*.ts`
- Tests: `src/**/*.test.ts`

## Never Do

_List critical rules for your project:_

- Never modify X without Y
- Never bypass validation in Z
- Never commit secrets to version control

---

## Example: E-commerce Platform Patterns

_This example shows how a real project might fill in this template._

Key decisions documented:
- State management: "Keep business logic in services. Keep controllers thin."
- Numeric precision: `Decimal` for monetary, `int` for counters, `float` for analytics ratios
- Critical pitfalls: N+1 queries, missing DB indexes, race conditions in inventory updates
