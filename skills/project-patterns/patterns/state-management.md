# State Management Patterns (Template)

> **Customize this file** with your project's state management decisions.

## Core Heuristic

_Define your project's philosophy for state management:_

> [Your heuristic here, e.g., "Keep business logic in services, data access in repositories"]

## State Location Decision Matrix

For each piece of state, explicitly decide where it lives:

| State Type | Location | Rationale |
|-----------|----------|-----------|
| User data | _Database_ | _Persistence required_ |
| Session state | _Redis/Cache_ | _Fast access, ephemeral_ |
| Computed values | _Memory_ | _Derived on demand_ |
| _Add your state types..._ | | |

## Layer Responsibilities

### Data Layer
_What belongs in your data/persistence layer:_
- Minimal representation for storage
- Schema definitions
- Migration logic

### Service Layer
_What belongs in your service/business layer:_
- Business logic and rules
- Validation beyond schema
- Orchestration between components

### API Layer
_What belongs in your API/presentation layer:_
- Request/response transformation
- Authentication/authorization
- Rate limiting

## Anti-Patterns

_Document state management mistakes specific to your project:_

### Don't: [Anti-pattern name]
```
// Bad example
```

### Do: [Correct pattern]
```
// Good example
```

## Related Files

_Link to key files in your codebase:_
- State definitions: `path/to/state/`
- Repositories: `path/to/repos/`
- Services: `path/to/services/`
