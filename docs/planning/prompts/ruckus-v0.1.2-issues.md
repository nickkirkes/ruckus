# Ruckus v0.1.2 — Issues to Address

## Overview

Ruckus v0.1.1 is tagged and public. This release addresses a breaking directory rename (do it now while user count is zero), pipeline correctness issues from the post-Phase 5 review, and high-priority token optimizations.

Read the full codebase before planning. Understand how skills reference each other, how agents reference shared files, and how setup creates project-level files.

---

## P0 — Breaking Change (must be first)

### Rename `docs/claude/` to `.ruckus/`

The current `docs/claude/` directory name is confusing for an open-source plugin — it's ambiguous, could clash with other tools, and doesn't communicate ownership. Rename to `.ruckus/`:

- `docs/claude/known-pitfalls.md` → `.ruckus/known-pitfalls.md`
- `docs/claude/.workflow-upgrades` → `.ruckus/workflow-upgrades` (drop the leading dot — already in a dotdir)
- Eliminate `docs/claude/CLAUDE.md` entirely — root `CLAUDE.md` is the only copy. Setup writes directly to root.
- Update every skill, agent, template, and doc file that references the old paths
- The upgrade skill needs a migration step: detect `docs/claude/` in existing projects and offer to move files to `.ruckus/`

---

## P1 — Pipeline Correctness

### Loop caps missing

Several pipeline loops have no bound and can run indefinitely:

- Stage 4: "2 consecutive NEEDS REVISION" can be bypassed by non-consecutive failures. Should be "2 total."
- Stage 5c: Subagent question-answering re-dispatch has no cap. Need 2-attempt cap with human escalation.
- Stage 6: Review-fix loop has no cap. Need 2-cycle cap matching verify-all's pattern.
- Stage 6: The "address warnings" gate option is ambiguous — could let the LLM loop without a human decision.

### Compaction preserve lists incomplete

Context compaction boundaries (added in Phase 4) risk losing critical data:

- Stage 4→5: Plan file path not in preserve list. Stage 5 could fail after compaction.
- Stage 5d: Task ID list not preserved. Orchestrator would need to re-read the plan.
- No compaction after Stage 7 before maturity checks — maturity checks run on bloated context.
- No re-validation of plan file path at Stage 5 start after Stage 4 compaction.

### Error handling ambiguity

- Quality check retry logic (Stage 5): "Fix the issue" doesn't distinguish auto-fixable (type error in a file this task modified) from unfixable (missing dependency, architecture problem). Agents guess.
- Review-plan with missing CLAUDE.md: Fails cryptically if setup wasn't run. Should return a clear error.
- Spec-reviewer reference: Any remaining references to "follow the checklist in spec-reviewer-prompt.md" are ambiguous — the runtime checklist is inlined, the file is reference-only.

---

## P2 — Maintenance & Optimization

### Agent preamble drift

- agent-preamble.md doesn't document why static-analysis and doc-writer have different preambles
- Implementer-prompt uses an abbreviated preamble that may have drifted from the canonical version
- No mechanism to detect preamble drift across agents during upgrade

### Audit-epic token budget

For epics with 10+ stories, all per-story reports accumulate in the orchestrator's context before synthesis (3-5K tokens). Should batch in groups of 5 and summarize between batches.

### Setup & upgrade hardening

- Required-field enforcement is behavioral only — no explicit gate between collecting answers and creating files
- Enrich mode has no definition of what constitutes a "gap" vs existing content
- `{{FORMATTER_COMMAND}}` can be left unreplaced if the agent misses the removal instruction
- Upgrade doesn't explicitly preserve user-added hooks when merging settings.json

### Documentation accuracy

- Token usage table doesn't reflect Phase 4 compaction savings
- Review vs review-plan relationship not explained
- No context management guidance for large builds (10+ tasks)
- fix/SKILL.md description says "self-upgrades" — should say "offers to create"
- Quick Start file list needs to reflect `.ruckus/` after rename
- "maturity" used in Quick Start without inline definition

---

## Explicitly Deferred (NOT in this release)

Do not include these — they are v0.2.0 candidates:

- Brand voice consistency across documentation
- Warmer SKILL.md preambles for solo devs
- Routing simple implementation tasks to Haiku
- Monorepo detection in setup
- Migration guide from predecessor workflows
- Stack-aware .claudeignore pruning during setup
- Upgrade sentinel comments in templates
- CLAUDE.md template example content
