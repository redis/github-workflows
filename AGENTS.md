# AGENTS.md

> This file is a map, not a manual. Start here, then follow pointers.

## Quick Start

1. Read this file (~100 lines)
2. Check `ARCHITECTURE.md` for system overview
3. Consult `docs/` for detailed guidance

## Repository Structure

```
AGENTS.md          # You are here - the entry point
ARCHITECTURE.md    # System overview, domains, package layering
docs/
├── design-docs/   # Design decisions and rationale
│   ├── index.md
│   └── core-beliefs.md
├── exec-plans/    # Execution plans for complex work
│   ├── active/
│   ├── completed/
│   └── tech-debt-tracker.md
├── product-specs/ # Product requirements and specifications
│   └── index.md
├── references/    # External documentation, LLM-friendly formats
└── generated/     # Auto-generated docs (schemas, API specs)
```

## Core Beliefs

Before making changes, internalize these principles:

1. **Context is scarce** — Keep documentation focused and discoverable
2. **Progressive disclosure** — Start simple, link to details
3. **Plans are artifacts** — Complex work gets versioned execution plans
4. **Docs follow code** — Documentation lives near what it describes
5. **Mechanical verification** — CI validates doc freshness and links

## How to Navigate

| I need to...                     | Go to                              |
|----------------------------------|------------------------------------|
| Understand the architecture      | `ARCHITECTURE.md`                  |
| Find a design decision           | `docs/design-docs/index.md`        |
| See active work plans            | `docs/exec-plans/active/`          |
| Review technical debt            | `docs/exec-plans/tech-debt-tracker.md` |
| Check product requirements       | `docs/product-specs/index.md`      |
| Find external API references     | `docs/references/`                 |
| See generated schemas            | `docs/generated/`                  |

## Working with Plans

### Small Changes
Use ephemeral, lightweight plans. No formal document required.

### Complex Work
1. Create an execution plan in `docs/exec-plans/active/`
2. Include: goal, approach, progress log, decision log
3. Move to `docs/exec-plans/completed/` when done
4. Update `tech-debt-tracker.md` if debt is introduced

## Documentation Standards

- **Freshness**: Docs must reflect current code behavior
- **Cross-linking**: Reference related docs explicitly
- **Ownership**: Each doc should have a clear scope
- **Verification**: CI checks for broken links and staleness

## When to Update This File

Update AGENTS.md only when:
- New top-level documentation categories are added
- Navigation patterns fundamentally change
- Core beliefs are revised

Do NOT add:
- Detailed implementation guidance (put in `docs/`)
- Temporary instructions (use execution plans)
- Project-specific context (use design docs)

## Validation

This repository uses mechanical checks:
- Link validation in CI
- Freshness checks for critical docs
- Doc-gardening automation for stale content

---

*Last verified: 2026-02-14*

