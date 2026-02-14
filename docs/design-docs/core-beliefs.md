# Core Beliefs

> Agent-first operating principles. These guide all design and implementation decisions.

## Context Management

### 1. Context is Scarce
- Every token in context has an opportunity cost
- Prefer pointers over inline content
- Keep files focused; split when they exceed ~200 lines
- Auto-generated content belongs in `docs/generated/`

### 2. Progressive Disclosure
- Start with the minimum viable context
- Teach agents where to look, not what to memorize
- Layer information: overview → details → implementation
- Use explicit "See also" links

### 3. Maps Over Manuals
- AGENTS.md is a table of contents, not an encyclopedia
- Navigation should be obvious and mechanical
- "I need to X" should map to exactly one location

## Documentation Philosophy

### 4. Docs Follow Code
- Documentation lives near what it describes
- When code changes, nearby docs must update
- Stale docs are worse than no docs

### 5. Plans Are Artifacts
- Complex work deserves versioned execution plans
- Progress and decisions are logged, not remembered
- Completed plans become searchable history

### 6. Mechanical Verification
- If a rule can be checked by CI, it should be
- Broken links, stale dates, missing cross-refs are failures
- Trust automation over discipline

## Implementation Principles

### 7. Explicit Over Implicit
- State assumptions in code and docs
- Prefer verbose clarity over clever brevity
- Magic is technical debt

### 8. Incremental Over Revolutionary
- Small, reversible changes compound
- Working code beats perfect plans
- Ship, learn, iterate

### 9. Ownership is Clear
- Every file has an owning domain
- Every domain has explicit boundaries
- Ambiguity creates gaps

## Anti-Patterns

Avoid these behaviors:

| Anti-Pattern | Why It Fails | Instead |
|--------------|--------------|---------|
| One giant instruction file | Crowds out task context | Use progressive disclosure |
| "Everything is important" | Nothing gets prioritized | Rank and filter |
| Undocumented decisions | Context lost between sessions | Write execution plans |
| Manual verification | Humans forget, drift happens | Automate checks |
| Copy-paste knowledge | Forks rot independently | Single source of truth |

---

*Status: ✅ Verified*
*Last verified: 2026-02-14*

