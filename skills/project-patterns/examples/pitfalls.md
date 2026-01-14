# Critical Pitfalls (Template)

> **Customize this file** with mistakes that have caused outages or data corruption in your project.

This document captures hard-won lessons - mistakes that seemed reasonable but caused serious problems. When implementing features, check this list.

## Categories

- **Data Integrity** - Mistakes that corrupt or lose data
- **Concurrency** - Race conditions and deadlocks
- **Performance** - Changes that cause severe slowdowns
- **Security** - Vulnerabilities introduced by common patterns
- **Operational** - Mistakes that cause outages or deployment issues

---

## Data Integrity

### Pitfall: [Name]

**What happened:** _Brief description of the incident_

**Why it seemed reasonable:** _Why someone might make this mistake_

**The problem:**
```
// Bad code example
```

**The fix:**
```
// Correct code example
```

**Detection:** _How to verify you haven't made this mistake_

---

## Concurrency

### Pitfall: [Name]

_Add concurrency pitfalls specific to your project..._

---

## Performance

### Pitfall: [Name]

_Add performance pitfalls specific to your project..._

---

## Security

### Pitfall: [Name]

_Add security pitfalls specific to your project..._

---

## Operational

### Pitfall: [Name]

_Add operational pitfalls specific to your project..._

---

## Checklist

Before merging, verify:

- [ ] Not introducing any of the pitfalls listed above
- [ ] Changes to [critical area X] reviewed by [specific team/person]
- [ ] Database migrations are reversible
- [ ] Feature flags in place for risky changes
- [ ] Monitoring/alerting updated if needed

---

## Adding New Pitfalls

When you discover a new pitfall:

1. Document the incident immediately while details are fresh
2. Include the bad code that caused it
3. Include the fix
4. Add detection/prevention guidance
5. Update the checklist if appropriate
