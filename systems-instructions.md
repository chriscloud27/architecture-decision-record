## ADR Authoring Guidance

Architectural Decision Records (ADRs) are required for architecture-level decisions.

- **Location**: ADRs live in `adr/`
- **Template**: Start from `adr/0000-template.md`
- **Write an ADR when a decision**:
    - Changes or establishes overall architecture (routing, auth, storage, tenancy model)
    - Chooses between technologies or platforms
    - Establishes a pattern that all future code must follow
    - Would be confusing or surprising to a new developer without context
    - Explicitly rejects a reasonable alternative
- **Do not write an ADR for**:
    - Implementation details
    - Bug fixes
    - Decisions that are obvious from reading the code
- **Numbering**: Use the next available four-digit number, e.g. `0008-title.md`
- **Status values**:
    - `proposed` (under discussion)
    - `accepted` (decided and active)
    - `deprecated` (no longer relevant)
    - `superseded by [ADR-XXXX](XXXX-title.md)` (replaced by a newer decision)

For full ADR policy and architecture context, see `CLAUDE.md`.