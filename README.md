# SmolSDD

## Why am I making this?

Existing toolkits haven't worked well for me:

- Speckit: [Github](https://github.com/github/spec-kit)
- Superpowers: [Github](https://github.com/obra/superpowers)
- OpenSpec: [Github](https://github.com/Fission-AI/OpenSpec)
- BMAD: [Github](https://github.com/bmad-code-org/BMAD-METHOD)

Common pain points:

- They try to be too one-stop shop.
  - That might be good for adoption by dabblers.
  - However, I've found it to be quite difficult start from these things as a base, actually understand the plumbing, and then customize it.
  - This is in spite of many of them billing themselves as "lightweight".
  - My own acid test: If your prompt toolkit comes with Javascript / Python glue code, it's not a SDD toolkit. It's a custom harness with holes, and it should not be billed as "lightweight".
- They tend to enforce opinionated flavors of SDD in the name of distinguishing themselves within the ecosystem, but using them often means needing to bend my brain to think the way their workflow wants me to.
- They bake in a lot of assumptions about model + harness capabilities, tend to encourage coupling to a particular model lab's feature set.
- **They leave you high and dry in the cases you where you learn something mid-implementation**.

## Goals

Shrinkwrapped, modularized spec-driven development toolkit.

- No setup scripts or glue code allowed.
- Institutional-wisdom / tribal-knowledge context needs to be injectable. This toolkit should contain only the _workflow_.
- No faff tokens. Agent-generated skills are a good start, but you can often do better than wholesale-dumping into your context window without additional consideration.
- Support Spec + Plan evolution.
  - It [often happens](https://www.linkedin.com/feed/update/urn:li:activity:7413956151144542208/) that you learn something mid-implementation which should change your plan.
  - SDD workflows need to track the story of that learning in a format that can be progressively discovered by future agent sessions.
- Project-specific "[constitutions](https://specdriven.ai/)" (encoded as technical-design docs). Different teams interact with the same piece of code in different ways. Agents working on a simple non-deterministic key-value store should not be laden with details about your strictly-sequencing message bus. Technical direction needs to be tailorable to the specific build. Technical direction needs to be tailorable to the specific build.
- Ralph-loop friendly?

## Features

- Uses beads for a persistent memory layer, internal to your agent swarm
- **Injectable tribal knowledge** via `project-patterns` skill (YOU customize this)
- Slash commands:
  - `spec` >> Create or iterate on a spec
  - `research` >> Get a research question answered
  - `techdesign` >> Create or iterate on a technical design doc
  - `plan` >> Create or iterate on a plan doc
  - `taskify` >> Create or iterate on the breakdown of implementation tasks for some plan
  - `implement` >> Complete or verify an implementation task
  - `handoff` >> Create a handoff for steered compaction to enable multi-session work
  - `takeon` >> Continue working on a handoff bead
- Agents (for parallel research in commands):
  - `codebase-locator` >> Find WHERE files live
  - `codebase-analyzer` >> Understand HOW code works
  - `codebase-pattern-finder` >> Find existing patterns to model after
  - `thoughts-locator` >> Find documents in thoughts/
  - `thoughts-analyzer` >> Extract insights from documents
  - `web-search-researcher` >> Research external resources
- Skills:
  - `beads-workflow` >> Core workflow state machine and patterns
  - `project-patterns` >> **Template** for YOUR team's tribal knowledge (customize this!)

## Quick Start

See [SETUP.md](./SETUP.md) for full installation and configuration instructions.

```bash
# 1. Copy to your project
cp -r smol-sdd/* your-project/.claude/

# 2. Initialize beads
cd your-project && bd init

# 3. Customize project-patterns skill with YOUR conventions
vim .claude/skills/project-patterns/SKILL.md

# 4. Create thoughts directory
mkdir -p thoughts/{specs,research,design,plans,handoffs}
```
