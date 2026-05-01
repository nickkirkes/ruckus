# E01 Epic Review

**Date:** 2026-04-24
**Reviewer:** Ruckus epic-reviewer (Opus)
**Verdict:** Ready

---

## Summary

All line numbers, file references, and behavioral claims verified against the actual codebase. Line budget arithmetic confirmed correct (fix/SKILL.md reaches 299/300 after S1-S4). Phasing is sound, ADR constraints are accurate, and no blockers found.

---

## Dimension Ratings

| Dimension | Rating |
|-----------|--------|
| Technical accuracy | PASS |
| Best practices | PASS |
| Risks | MINOR CONCERN |
| Overengineering | PASS |
| AC quality | PASS |
| Dependencies | PASS |

---

## Findings

### Technical Accuracy — PASS

- All 30+ line references verified correct against current file content
- S1 file count (18 files) matches bullet list
- Baseline line counts confirmed: build=289, fix=294, setup=165, upgrade=117, review-plan=90, audit-epic=129
- S1 grep-based AC scope correctly excludes `docs/adrs/` (ADRs retain old paths with footnotes)
- Cumulative line budget: S2 (+1) + S3 (+3) + S4 (+1) = +5 lines → fix at 299, build at 294

### Best Practices — PASS

- ADR compliance confirmed for all 8 stories (zero conflicts)
- 300-line limit discipline with compression strategies specified per story
- ADR footnote approach (S1) is appropriate — historical path references, not decision changes
- `{{PLACEHOLDER}}` convention and `disable-model-invocation: true` correctly applied

### Risks — MINOR CONCERN

**Risk 1 — S1 upgrade migration path completeness:**
The migration step moves files from `docs/claude/` to `.ruckus/` but does not update stale `docs/claude/known-pitfalls.md` paths that may exist in the user's root CLAUDE.md (generated from the old template). Low priority given zero users, but would make migration more thorough.

**Suggestion:** Add to upgrade migration step: "After moving files, update the `Known pitfalls:` path in root CLAUDE.md from `docs/claude/known-pitfalls.md` to `.ruckus/known-pitfalls.md` if the old path appears."

**Risk 2 — S1 setup/SKILL.md blast radius:**
S1 touches ~30 of 165 lines across 7 steps in setup/SKILL.md. The epic provides clear current/target behavior for key changes, which mitigates this, but it's the highest-risk single-file change in the epic.

**Risk 3 — 1-line headroom on fix/SKILL.md:**
Acknowledged in the epic. Mitigated by specifying exact target text for each change. S4 AC 6 already includes explicit line count verification.

### Overengineering — PASS

- S5 preamble drift detection: small (6 lines across 3 files), legitimate safeguard
- S6 token budget batching: lightweight conditional, reasonable 10-story threshold
- S7 setup hardening: all 4 changes address specific observed behavioral gaps
- S3 compaction: targeted, small changes addressing concrete context-loss risks

### AC Quality — PASS

- S1: 12 ACs, all specific and testable; grep verification AC is particularly strong
- S2: 4 ACs, all testable by text inspection
- S4: AC 5 (spec-reviewer verification-only) correctly marked as no-implementation
- S7: AC 5 (regression prevention) is good practice

**Suggestion (S3 AC 4):** Rephrase from "Each compaction preserve list includes exactly the data its consuming stage needs" to: "Stage 5d preserve list includes task ID list. Post-Stage 7 compaction preserves feature summary, files changed, and verification verdict." Makes it mechanically verifiable.

### Dependencies — PASS

- Phase ordering verified correct
- S1 must be first (all others use `.ruckus/` paths)
- Phase 2 (S2+S5+S6): confirmed disjoint file sets
- Phase 3 (S3+S7): confirmed disjoint; upgrade/SKILL.md correctly sequenced after S5
- Phase 4 (S4+S8): fix/SKILL.md overlap in different sections (frontmatter vs Stage 5c) — safe
- No hidden dependencies found

---

## Recommendations

1. **(S1, minor)** Add stale-path update to upgrade migration step for user's root CLAUDE.md
2. **(S3, minor)** Make AC 4 mechanically verifiable by listing specific data items per compaction point
3. **(General, commendation)** "Issues Resolved During Planning" section (8 items) demonstrates unusually thorough pre-implementation validation
