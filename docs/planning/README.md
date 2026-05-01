# Roughly — Planning Index

**Last Updated:** 2026-04-30
**Current Release:** v0.1.4 (released 2026-04-30 — E02 ruckus → roughly rename complete; tagged `v0.1.4`; GitHub repo renamed to `nickkirkes/roughly`)
**Plugin Version:** 0.1.4

---

## Release Status

| Version | Title | Status | Released | Notes |
|---------|-------|--------|----------|-------|
| v0.1.0 | Initial public release | Complete | 2026-04-22 | 9 skills, 7 agents |
| v0.1.1 | Plan mode fix + review-plan dispatch | Complete | 2026-04-24 | Patch release |
| v0.1.2 (E01) | Directory Rename + Pipeline Hardening | Complete | 2026-04-27 | Breaking change; audit 42/45 ACs MET, 3 PARTIAL cosmetic |
| v0.1.3 | E01 audit follow-up (upgrade-skill agent-installation fix) | Complete | 2026-04-28 | Single-fix patch |
| v0.1.4 (E02) | Rename: ruckus → roughly | Complete | 2026-04-30 | Hard-cut rename, no aliases. 7 stories, 4 PRs (S2.4 folded into S2.2 PR #19). GitHub repo renamed `ruckus` → `roughly`. v0.1.4 tag pushed; GitHub release created. |
| v0.2.0 | TBD | Not Started | — | See Deferred Items below for candidate scope |

---

## Completed Epics

### E02: Rename ruckus → roughly (v0.1.4)

All 7 stories merged. v0.1.4 tagged 2026-04-30. GitHub repo renamed to `nickkirkes/roughly`. See [epics/complete/E02-rename-roughly.md](epics/complete/E02-rename-roughly.md), [v1 review](epics/complete/E02-rename-roughly-review.md), and [v2 review](epics/complete/E02-rename-roughly-review-v2.md).

| Story | Name | Status | PR |
|-------|------|--------|-----|
| S2.1 | Plugin identity and slash-command namespace | Complete | #18 (merged 2026-04-29) |
| S2.2 | Skills `.ruckus/` migration, version-line rename, skill prose, v0.1.4 upgrade-migration step | Complete | #19 (merged 2026-04-29; S2.4 templates folded in as P1 follow-up; line-budget fix-up `b2fa658` post-merge) |
| S2.3 | Agent preamble sync (`.ruckus/` → `.roughly/`) | Complete | #20 (merged 2026-04-30; includes Stop-hook scope expansion) |
| S2.4 | Templates rename | Complete | Folded into PR #19 (commit `a8e1070`) |
| S2.5 | Documentation prose, ADR footnotes | Complete | #21 (merged 2026-04-30) |
| S2.6 | This repo's own `.ruckus/` → `.roughly/` dogfood | Complete | #20 (merged 2026-04-30) |
| S2.7 | Final verification, version bump, CHANGELOG, tag | Complete | #22 (merged 2026-04-30; caught + fixed 2 S2.5 ADR-prose oversights via post-validation audit) |

**Validation history:** v1 epic draft validated 2026-04-28 by 12 parallel agents (3 reviewers × 4 flagged stories). v2 review (Opus epic-reviewer) returned `Needs Revision` with 10 prioritized recommendations — all addressed in v2. Second Opus pass on v2 returned `Ready` with 3 narrow corrections — all 3 plus 4 lower-severity issues addressed in v3 before S2.1 launch.

**Scope expansions accepted during execution:**
- Pre-flight migration check added to 6 skills (build, fix, review, review-plan, review-epic, setup) — protects v0.1.3 users who install the new plugin without first running `/roughly:upgrade`. PR #19.
- Structural Stop-hook (`.claude/hooks/verify-all.sh`) added — non-blocking drift detection on stale paths, skill/agent line caps, and HTML comment integrity. Aligns with ADR-005 `stop-hook-v1`. PR #20.
- Two pitfalls captured during S2.3 wrap-up (PR #20) and two during S2.5 wrap-up (PR #21) — on-pattern Stage 8 knowledge-loop output.

### E01: Directory Rename + Pipeline Hardening (v0.1.2)

All 8 stories merged, audit complete. See [epics/complete/E01-directory-rename-pipeline-hardening.md](epics/complete/E01-directory-rename-pipeline-hardening.md) and [audit](epics/complete/E01-directory-rename-pipeline-hardening-audit.md).

| Story | Name | Status |
|-------|------|--------|
| S1 | Rename `docs/claude/` to `.ruckus/` | Complete |
| S2 | Pipeline loop caps | Complete |
| S3 | Compaction preserve lists and re-validation | Complete |
| S4 | Error handling disambiguation | Complete |
| S5 | Agent preamble drift documentation and detection | Complete |
| S6 | Audit-epic token budget batching | Complete |
| S7 | Setup and upgrade hardening | Complete |
| S8 | Documentation accuracy | Complete |

**Post-tag fix (v0.1.3):** Audit found and corrected an upgrade-skill bug that would have offered to install plugin-shipped agents into user projects. Released as v0.1.3 on 2026-04-28.

---

## Active Blockers

None. v0.1.4 shipped 2026-04-30.

---

## Next Actions

No active engineering work queued. The plugin is at v0.1.4 on `main` with the GitHub repo renamed to `nickkirkes/roughly`. Consider scoping v0.2.0 from the Deferred Items list when ready for the next planning cycle.

---

## Deferred Items

### Resolved during E02 (no longer deferred)

- ~~GitHub repo rename `ruckus` → `roughly`~~ — completed 2026-04-30 alongside v0.1.4 tag push.

### v0.1.4 follow-ups still active (low priority)

- **`roughly.dev` site:** marketing surface where the `~roughly` brand wordmark lives. Out of repo scope; tracked separately.
- **External links:** any third-party docs, posts, blog references, or social-media mentions of the old `ruckus` name. The repo's own redirect from `nickkirkes/ruckus` (preserved by GitHub on rename) handles incoming clones, but third-party content is out of automated reach.

### Pushed to v0.2.0 (carried over from the v0.1.2 issues list)

- Brand voice consistency across documentation
- Warmer SKILL.md preambles for solo devs
- Routing simple implementation tasks to Haiku
- Monorepo detection in setup
- Migration guide from predecessor workflows
- Stack-aware .claudeignore pruning during setup
- Upgrade sentinel comments in templates
- CLAUDE.md template example content
