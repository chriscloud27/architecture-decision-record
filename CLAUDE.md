# Claude Code — Project Guide

Guidelines for working with Claude Code in this repository.

---

## Architectural Decision Records (ADRs)

ADRs live in [`adr/`](adr/). Use [`adr/0000-template.md`](adr/0000-template.md) as the starting point.

### When to write an ADR

Write a new ADR when a decision:
- Changes or establishes the overall architecture (routing, auth, storage, tenancy model)
- Chooses between technologies or platforms (e.g. Firebase vs Supabase)
- Establishes a pattern that all future code must follow (e.g. RLS on every table)
- Would be confusing or surprising to a new developer without context
- Explicitly rejects a reasonable alternative — document why

Do **not** write an ADR for:
- Implementation details (which component to use, how to name a variable)
- Bug fixes
- Decisions that are obvious from reading the code

### Numbering
Use the next available four-digit number: `0005-title.md`, `0006-title.md`, etc.

### Status values
- `proposed` — under discussion
- `accepted` — decided and active
- `deprecated` — no longer relevant
- `superseded by [ADR-XXXX](XXXX-title.md)` — replaced by a newer decision
