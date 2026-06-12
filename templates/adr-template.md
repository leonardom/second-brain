# Architecture Decision Record (ADR) Template

Use this template to document significant architectural decisions. ADRs capture the context, decision, and consequences at the time a decision is made. Treat them as immutable records — if a decision changes, create a new ADR and reference the old one.

**File naming:** `NNNN-short-descriptive-title.md` (e.g., `0001-use-postgresql-as-primary-database.md`)

**Location:** Store ADRs close to the code they affect (e.g., `docs/adr/`) or here in the templates directory.

---

# ADR-NNNN: [Short Title Describing the Decision]

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded by [ADR-XXXX](./XXXX-title.md)
**Deciders:** [List names or team]
**Tags:** #scalability #reliability #security #dx #tradeoffs

---

## Context

> Describe the issue motivating this decision and any context that influenced it.
> Include:
> - What problem are we solving?
> - What forces / constraints are at play? (technical debt, team size, deadlines, regulatory requirements)
> - What alternatives were considered?

_Example: Our monolithic application has reached a point where independent deployability and team autonomy require us to split into separate services. We have 4 product teams and deploy 10+ times per day._

---

## Decision

> State the decision clearly and concisely. Use an active voice.
> _"We will use X."_ or _"We have decided to adopt Y."_

_Example: We will adopt a microservices architecture, splitting the monolith into services aligned with our 4 bounded contexts: Orders, Inventory, Users, and Payments._

---

## Rationale

> Explain **why** this decision was made over the alternatives.
> - Why is this the right approach given the context?
> - What are the key benefits?
> - Which alternatives were rejected and why?

### Option A: [Chosen option] ✅
- Pro: ...
- Pro: ...
- Con: ...

### Option B: [Alternative considered] ❌
- Pro: ...
- Con: ... (reason for rejection)

### Option C: [Alternative considered] ❌
- Con: ... (reason for rejection)

---

## Consequences

> Describe the resulting context after applying the decision.
> Include both positive and negative consequences, and any follow-up actions required.

### Positive
- ...

### Negative / Trade-offs
- ...

### Neutral
- ...

### Follow-up Actions
- [ ] Action item 1 (Owner: @name, Due: YYYY-MM-DD)
- [ ] Action item 2 (Owner: @name, Due: YYYY-MM-DD)

---

## Related Decisions

- [ADR-XXXX: Related decision title](./XXXX-related-title.md)

---

## References

- Link to relevant documentation, RFC, or external article
- Link to spike/proof-of-concept branch or PR

---

*Template based on [Michael Nygard's ADR format](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) and [MADR (Markdown Architectural Decision Records)](https://adr.github.io/madr/).*
