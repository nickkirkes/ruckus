# Architecture Decision Records

ADRs capture the reasoning behind significant design decisions in Roughly. They exist so contributors can understand *why* the code is structured a certain way — not just *how*.

## Numbering

`ADR-NNN-short-slug.md` — sequential, zero-padded to three digits. Never reuse a number; deprecated ADRs keep their number and get a `Deprecated` status.

## Status Vocabulary

- **Accepted** — current and enforced
- **Deprecated** — no longer applies; kept for historical context
- **Superseded by ADR-NNN** — replaced by a newer decision

## What Warrants an ADR

Write an ADR when a decision:
- Changes the pipeline stage structure (number, order, or gating behavior)
- Changes how subagents are dispatched, scoped, or coordinated
- Changes what `disable-model-invocation: true` applies to
- Removes or fundamentally alters a review/verification mechanism
- Would surprise a contributor who read the existing ADRs

Bug fixes, template improvements, documentation changes, and new agent definitions do *not* need ADRs unless they alter the behaviors above.
