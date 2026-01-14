# Testing Patterns (Template)

> **Customize this file** with your project's testing conventions.

## Test Commands

_Document the commands used in your project:_

| Purpose | Command | Notes |
|---------|---------|-------|
| Unit tests | `npm test` | _Runs jest/vitest/etc_ |
| Integration tests | `npm run test:integration` | _Requires DB setup_ |
| E2E tests | `npm run test:e2e` | _Requires full stack_ |
| Lint | `npm run lint` | _ESLint/Prettier_ |
| Type check | `npm run typecheck` | _TypeScript compilation_ |
| All checks | `npm run validate` | _Full CI pipeline locally_ |

## Test Requirements by Change Type

_Define what tests are required for different types of changes:_

### API Changes
- [ ] Unit tests for handler logic
- [ ] Integration tests for DB operations
- [ ] API contract tests

### Business Logic Changes
- [ ] Unit tests for new logic
- [ ] Edge case coverage
- [ ] Regression tests for related features

### Database Changes
- [ ] Migration tests
- [ ] Rollback verification
- [ ] Data integrity tests

## Test File Conventions

_Document where tests live:_

```
src/
├── services/
│   ├── userService.ts
│   └── userService.test.ts    # Co-located unit tests
└── __tests__/
    └── integration/           # Integration tests
        └── user.integration.test.ts
```

## Mocking Patterns

_Document how to mock dependencies:_

### External Services
```typescript
// Example mock pattern
```

### Database
```typescript
// Example mock pattern
```

## Test Data Patterns

_Document how to set up test data:_

### Fixtures
- Location: `test/fixtures/`
- Usage: Import and use in tests

### Factories
- Location: `test/factories/`
- Usage: Generate dynamic test data

## BDD/Feature Tests

_If using BDD, document the conventions:_

- Feature files: `features/*.feature`
- Step definitions: `features/steps/*.ts`
- Running: `npm run test:bdd`

## Coverage Requirements

_Define minimum coverage thresholds:_

| Metric | Minimum |
|--------|---------|
| Lines | 80% |
| Branches | 75% |
| Functions | 80% |
