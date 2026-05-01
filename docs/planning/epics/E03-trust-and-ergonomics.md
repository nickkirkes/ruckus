# E03 — Trust hardening + ergonomics + CI

**Status:** Drafted (PM handoff for v0.1.5)
**Target version:** v0.1.5
**Target effort:** 6-7 wk
**Dependencies:** E01 (pipeline foundation, audit follow-up shipped v0.1.3); E02 (rename to `roughly` shipped v0.1.4 — namespace, dotdir, version-line identifier all assumed in place)

---

## Release thesis

Roughly's value is enforcement. Enforcement with known holes is theater. v0.1.5 closes the silent-failure modes still present in the pipeline — plan-mode hijack, untested maturity gaps, ambiguous abort UX — and lays the regression-coverage groundwork (plugin self-test CI) that every subsequent release will depend on.

Three clusters: **trust hardening** (S0–S6), **ergonomics** (S7–S10), **CI** (S11). Plus **docs** (S12), which lands incrementally rather than as a final-week dump.

Scope is frozen. Items surfaced during epic writing that are clearly related but out of scope are listed under [v0.1.6 candidates](#v016-candidates).

---

## Risk register

1. **Plan-mode detection mechanism uncertainty (S1).** Plan mode is signalled to the orchestrator via a `<system-reminder>` block and `ExitPlanMode` tool availability — neither is a programmatic API a skill can introspect. The mechanism choice (preamble guard, hook, or both) is gated on the S0 spike. If the spike finds no reliable signal, S1 falls back to preamble-only and accepts a documented edge case where mid-pipeline auto-engagement bypasses the guard. Risk: enforcement still has a known hole post-v0.1.5; mitigated by the fallback being explicit, not silent.

2. **CI bootstrapping (S11).** The plugin tests itself against the repo containing the plugin. A naive run mutates `.roughly/`, writes plan files in `docs/plans/`, and may dirty the working tree. Self-test must run in an ephemeral worktree/checkout with strict teardown, or CI poisons source-repo state visible to subsequent commits. Risk: a CI run that "passes" but corrupts the dogfood `.roughly/` state masks bugs; mitigated by isolation contract in S11a.

3. **Docs scope creep (S12).** "roughly.dev v0.1.5 floor" is four pages, but the underlying surface area is 9 (10 with `/roughly:help`) skills + 7 agents + 8 ADRs. Without explicit per-page outlines and word budgets in the story, docs balloons and slips the release. Risk: docs becomes a final-week landmine; mitigated by docs stories laddering throughout the release rather than batch-landing.

4. **Retry-loop tuning regressions (S10).** Raising caps on cheap checks can hide flakiness; replacing hard escalation with prompts shifts cost to humans mid-pipeline. Each adjustment needs a before/after dogfood pass on a known case. Risk: silent trust degradation; mitigated by per-cap rationale recorded inline and dogfood verification gated by S11 CI.

5. **Stop-hook-v1 templating completion (S2).** This repo's [.claude/hooks/verify-all.sh](.claude/hooks/verify-all.sh) is a dogfood instance with project-specific drift checks (line caps for `agents/`, `.ruckus/` legacy detection, etc.) — it is not a plugin-shipped template. The maturity check must template a generic Stop hook into the user's `.claude/`, handling the case where the user already has a Stop hook configured (merge vs prompt vs decline). Risk: under-spec'd templating ships a hook that conflicts with an existing user hook; mitigated by explicit conflict-handling AC in S2.

---

## Stories

Stories are grouped by cluster. Sequencing — which is by dependency, not roadmap order — appears in [the final section](#sequencing).

### Trust hardening cluster

#### E03.S0: Plan-mode detection spike

**Maps to roadmap item:** #1 (gates S1)
**Type:** Investigation/spike (½-day timebox)

**Files touched:**
- `docs/planning/spikes/plan-mode-detection-findings.md` (new — scratch output, not committed if conclusions land in S1)

**Context:**

S1 needs a detection mechanism, but it's unclear what programmatic signals are available to a skill at runtime. This spike is bounded research: probe what the harness exposes, decide preamble-only vs preamble+hook, and document the answer before S1 starts. Half-day cap.

**Acceptance criteria:**
- [ ] Findings doc enumerates plan-mode signals observable to a skill at runtime (system reminder text, ExitPlanMode tool availability, any others)
- [ ] Findings doc identifies what triggers Claude Code's plan-mode auto-engagement (SessionStart hooks, settings, user toggles) — confirmed empirically, not from documentation alone
- [ ] Findings doc reports whether `ExitPlanMode` invoked from a skill body reliably exits plan mode, with at least one dogfood test result attached
- [ ] Findings doc lists which Claude Code hook events (SessionStart, PreToolUse, Stop, others) fire under plan mode and which do not
- [ ] Findings doc concludes with one of: **preamble-only** (default), **preamble + lightweight hook** (only if auto-engagement bypasses preamble), or **inconclusive — implementation must accept documented edge case** — with rationale
- [ ] No "both as belt-and-suspenders" outcome unless explicitly justified by a failure case observed during the spike

**Verification:**
- Spike output reviewed before S1 starts; conclusion drives S1's mechanism choice
- If timebox exceeded without conclusion, spike returns "inconclusive" and S1 proceeds preamble-only with a known-pitfalls.md entry documenting the gap

**Dependencies:** None — ships first.

**Out of scope:**
- Implementing the detection mechanism (that's S1)
- Writing tests for detection beyond what's needed to validate the spike's conclusion

---

#### E03.S1: Plan-mode auto-detect/exit at Stage 1 of build/fix

**Maps to roadmap item:** #1 (highest-value item in v0.1.5)

**Files touched:**
- [skills/build/SKILL.md](../../skills/build/SKILL.md) — preamble + Stage 1
- [skills/fix/SKILL.md](../../skills/fix/SKILL.md) — preamble + Stage 1
- [.roughly/known-pitfalls.md](../../.roughly/known-pitfalls.md) — detection contract (extend existing plan-mode pitfall at L14)
- Possibly `.claude/hooks/<name>.sh` (new) — only if S0 concludes a hook is required

**Context:**

The plan-mode hijack is documented in [.roughly/known-pitfalls.md:14](../../.roughly/known-pitfalls.md#L14): when plan mode is active during a `/roughly:build` or `/roughly:fix` invocation, plan-mode's workflow substitutes for Stages 1–4 and Stage 4's `/roughly:review-plan` dispatch is silently skipped. Without S1, ADR-001 (plan verification as blocking subagent) is unenforced — the build skill's review-plan call never fires.

S1 commits to **observable behavior only**, not a specific mechanism. Mechanism is chosen in S0.

**Acceptance criteria:**
- [ ] When `/roughly:build` is invoked while Claude Code's plan mode is active, Stage 1 does not begin until plan mode is exited
- [ ] When `/roughly:fix` is invoked while Claude Code's plan mode is active, Stage 1 does not begin until plan mode is exited
- [ ] On detection, the orchestrator either invokes `ExitPlanMode` and continues into Stage 1, or aborts with a one-line redirect message (the choice depends on S0 findings)
- [ ] The detection contract is documented in [skills/build/SKILL.md](../../skills/build/SKILL.md) preamble and synced verbatim to [skills/fix/SKILL.md](../../skills/fix/SKILL.md) preamble — manual sync as with [agents/agent-preamble.md](../../agents/agent-preamble.md), no automation
- [ ] If a hook ships per S0 findings, the hook is added to [skills/setup/templates/settings.json.template](../../skills/setup/templates/settings.json.template) and templated into user projects on `/roughly:setup`
- [ ] Known-pitfalls.md L14 entry is updated to reflect the new enforcement path; the silent-failure mode is recategorized as "blocked by S1 enforcement" not "open hole"

**Verification:**
- Manual dogfood: invoke `/roughly:build` from a session where plan mode is active before the command runs; confirm Stage 1 does not enter without an exit step
- Manual dogfood: same test for `/roughly:fix`
- CI dogfood (post-S11): scripted scenario asserts the abort/exit behavior under plan mode
- Re-read `.roughly/known-pitfalls.md` entry to confirm the rewrite is accurate

**Dependencies:** S0 (mechanism conclusion).

**Out of scope:**
- Detection of the generic `Plan` agent dispatch in non-plan-mode sessions (different concern; not a hijack)
- Hardening against hostile users who manually re-enter plan mode mid-pipeline (out of v0.1.5's threat model)

---

#### E03.S2: Stop-hook-v1 maturity check completion

**Maps to roadmap item:** #2

**Files touched:**
- [skills/build/SKILL.md](../../skills/build/SKILL.md) — Stage 8 maturity check section (`stop-hook-v1` block at L268)
- [skills/fix/SKILL.md](../../skills/fix/SKILL.md) — Stage 8 maturity check section (`stop-hook-v1` block at L271)
- [skills/setup/SKILL.md](../../skills/setup/SKILL.md) — initial setup offer (currently absent)
- `skills/setup/templates/verify-all-stop-hook.sh.template` (new — generic Stop hook template, **not** the dogfood [.claude/hooks/verify-all.sh](../../.claude/hooks/verify-all.sh) which has roughly-repo-specific checks)
- [skills/setup/templates/settings.json.template](../../skills/setup/templates/settings.json.template) — Stop hook entry

**Context:**

`stop-hook-v1` exists as a maturity check stub in build/fix Stage 8 today: "If `.claude/settings.json` has no `Stop` hook AND verify-all has 2+ meaningful checks AND not declined, offer." But there is no template for the hook itself — accepting "yes" today does nothing because the orchestrator has no script to template.

S2 closes the gap: ship a generic Stop hook template, plumb it through setup and the maturity check, and handle conflict with existing user hooks. The dogfood [.claude/hooks/verify-all.sh](../../.claude/hooks/verify-all.sh) stays as-is (project-specific drift checks for the plugin's own development); this story produces a separate, project-agnostic template.

**Acceptance criteria:**
- [ ] New template `skills/setup/templates/verify-all-stop-hook.sh.template` exists, runs verify-all commands from CLAUDE.md (or `.roughly/commands.md` if present), and reports drift via `systemMessage` JSON when checks fail
- [ ] The template is project-agnostic — no hard-coded paths, line caps, or `.ruckus/` legacy strings; placeholders used where setup must inject project specifics
- [ ] [skills/setup/SKILL.md](../../skills/setup/SKILL.md) gains a `stop-hook-v1` offer in the initial setup flow (currently only in build/fix wrap-up), gated on the same condition as build/fix
- [ ] When the user accepts `stop-hook-v1`, the template is copied to `.claude/hooks/<name>.sh` and the `Stop` entry added to the user's `.claude/settings.json`
- [ ] If `.claude/settings.json` already has a `Stop` hook, the orchestrator prompts: keep existing / replace / merge (chained execution) / decline — no silent overwrite
- [ ] Acceptance is recorded as `stop-hook-v1-added YYYY-MM-DD` in `.roughly/workflow-upgrades`; decline is recorded as `stop-hook-v1-declined`
- [ ] The build/fix Stage 8 `stop-hook-v1` offer text is updated to reflect the new templating path (no longer a no-op)

**Verification:**
- Dogfood `/roughly:setup` in a fresh project that has no `.claude/settings.json`; accept the offer; confirm hook file written, settings entry added, and `.roughly/workflow-upgrades` updated
- Dogfood `/roughly:setup` in a project that already has a `Stop` hook; confirm the conflict prompt appears and each branch behaves correctly
- Manually trigger the templated hook (`bash .claude/hooks/<name>.sh`) and confirm it reports drift correctly when verify-all fails

**Dependencies:** S3 (retire test-verify-v1 / pitfalls-organized-v1) lands first to avoid double-touching the maturity check section.

**Out of scope:**
- Replacing the dogfood [.claude/hooks/verify-all.sh](../../.claude/hooks/verify-all.sh) with the templated version
- Bumping `stop-hook-v1` to v2 (no change of behavior justifies a version bump per ADR-005)
- Cross-platform Stop hook support beyond bash (Windows users out of v0.1.5 threat model)

---

#### E03.S3: Retire test-verify-v1 and pitfalls-organized-v1

**Maps to roadmap item:** #3

**Files touched:**
- [skills/build/SKILL.md:260-269](../../skills/build/SKILL.md#L260-L269) — remove `pitfalls-organized-v1` and `test-verify-v1` blocks
- [skills/fix/SKILL.md:263-271](../../skills/fix/SKILL.md#L263-L271) — same
- [agents/doc-writer.md](../../agents/doc-writer.md) — fold trigger logic into known-pitfalls write path
- [docs/adrs/ADR-005-versioned-maturity-checks.md](../../docs/adrs/ADR-005-versioned-maturity-checks.md) — footnote noting retirement (not a new ADR)

**Context:**

These two checks fire at Stage 8 wrap-up of every build/fix run. `pitfalls-organized-v1` triggers when known-pitfalls.md exceeds 80 lines; `test-verify-v1` triggers when test config exists but verify-all's test step is a placeholder. Both are work that the doc-writer agent — which already runs after every successful build/fix — could perform contextually as part of its known-pitfalls write path.

Retiring them from the maturity-check loop simplifies wrap-up, removes nag patterns, and centralizes pitfall hygiene. The risk is coverage loss: doc-writer fires only on pipeline-driven writes, not on manual edits to `.roughly/known-pitfalls.md`. See [open questions](#open-questions).

**Acceptance criteria:**
- [ ] `pitfalls-organized-v1` and `test-verify-v1` blocks removed from [skills/build/SKILL.md](../../skills/build/SKILL.md) and [skills/fix/SKILL.md](../../skills/fix/SKILL.md) maturity check sections
- [ ] [agents/doc-writer.md](../../agents/doc-writer.md) gains an organize-suggestion step: when about to write to known-pitfalls.md, if the post-write file would exceed 80 lines, append a one-line note suggesting reorganization
- [ ] [agents/doc-writer.md](../../agents/doc-writer.md) gains a verify-all test integration suggestion: when test config is detected (presence of `package.json` test script, `pytest.ini`, etc.) and the verify-all test command is a placeholder per CLAUDE.md, append a one-line note suggesting adding the test step
- [ ] [docs/adrs/ADR-005-versioned-maturity-checks.md](../../docs/adrs/ADR-005-versioned-maturity-checks.md) gains a footnote noting `pitfalls-organized-v1` and `test-verify-v1` were retired in v0.1.5 (formal retirement, not a version bump per the ADR's reasoning)
- [ ] Existing entries in `.roughly/workflow-upgrades` for these check IDs are not auto-cleaned (they remain as historical record)
- [ ] No skill body grows past 300 lines as a result of changes (verify per CLAUDE.md cap)

**Verification:**
- Dogfood `/roughly:build` on a project where known-pitfalls.md is 85 lines; confirm doc-writer's note appears in the wrap-up summary, NOT a Stage 8 maturity-check prompt
- Dogfood `/roughly:fix` on a project with `package.json` test script and placeholder verify-all test step; confirm doc-writer's suggestion appears
- `wc -l skills/build/SKILL.md skills/fix/SKILL.md` ≤ 300 each

**Dependencies:** None blocking; sequenced before S2 to avoid double-touching maturity check sections.

**Out of scope:**
- Bumping `investigator-v1` or `stop-hook-v1` versions
- Adding new maturity checks to replace these
- Cleaning up historical `pitfalls-organized-v1-declined` entries from existing user `.roughly/workflow-upgrades` files

---

#### E03.S4: Pre-flight migration check in remaining 2 skills

**Maps to roadmap item:** #4

**Files touched:**
- [skills/audit-epic/SKILL.md](../../skills/audit-epic/SKILL.md) — preamble (mirror existing pattern from build/fix L19)
- [skills/verify-all/SKILL.md](../../skills/verify-all/SKILL.md) — preamble (same)

**Context:**

6 of 9 skills currently abort with a redirect to `/roughly:upgrade` if legacy `.ruckus/` state is detected (build, fix, review, review-plan, review-epic, setup). The roadmap line "remaining 3" is imprecise — `/roughly:upgrade` is the migration target, not a redirect candidate, so the actual scope is 2 skills: audit-epic and verify-all. Marker-aware resume improvements within `/roughly:upgrade` are surfaced as a [v0.1.6 candidate](#v016-candidates).

ROADMAP.md item 4 wording is updated separately as part of this story.

**Acceptance criteria:**
- [ ] [skills/audit-epic/SKILL.md](../../skills/audit-epic/SKILL.md) preamble contains the standard pre-flight migration check: "If `.ruckus/.migration-in-progress`, `.ruckus/known-pitfalls.md`, or `.ruckus/workflow-upgrades` exists, abort with: 'Legacy `.ruckus/` state detected... Run `/roughly:upgrade` to migrate or resume, then re-run.' A `.ruckus/` directory containing only user-extras (post-`leave` state from a completed upgrade) is fine — proceed."
- [ ] [skills/verify-all/SKILL.md](../../skills/verify-all/SKILL.md) preamble contains the same check
- [ ] Wording is identical across all 8 skills that now have the check (audit-epic, build, fix, review, review-plan, review-epic, setup, verify-all) — drift detected by [.claude/hooks/verify-all.sh](../../.claude/hooks/verify-all.sh) if added as a check
- [ ] [docs/ROADMAP.md](../../docs/ROADMAP.md) item 4 wording corrected to "Pre-flight migration check in remaining 2 skills (currently 6/9, upgrade excluded by design)"
- [ ] No skill body grows past 300 lines

**Verification:**
- `rg -n "Legacy \`.ruckus/\` state detected" skills/` returns matches in 8 skills (build, fix, review, review-plan, review-epic, setup, audit-epic, verify-all)
- Manual dogfood: create a fake `.ruckus/.migration-in-progress` file; invoke `/roughly:audit-epic` and `/roughly:verify-all`; confirm both abort with the redirect message

**Dependencies:** None; independent of pipeline changes.

**Out of scope:**
- Marker-aware resume improvements in [skills/upgrade/SKILL.md](../../skills/upgrade/SKILL.md)
- Adding pre-flight to `/roughly:help` (S8 will define this on its own)

---

#### E03.S5: Document Edit `replace_all` dual-semantic-token failure

**Maps to roadmap item:** #5

**Files touched:**
- [CONTRIBUTING.md](../../CONTRIBUTING.md) — new "Tooling pitfalls" section (or equivalent)

**Context:**

The S02.7 wrap-up incident (recorded at [.roughly/known-pitfalls.md:38](../../.roughly/known-pitfalls.md#L38)) showed that `Edit`'s `replace_all: true` is dangerous when the same token serves dual semantic roles in a file — legacy detection code and user-facing prose both contained "ruckus" in [.claude/hooks/verify-all.sh](../../.claude/hooks/verify-all.sh), and a bulk replacement silently inverted the legacy detector. This is a contributor pitfall, not a runtime issue — prose-only documentation is sufficient.

Code-level defense is explicitly deferred per the roadmap's "Deferred" section ("Edit replace_all code-level defense: prose-only in v0.1.5; code defense waits for a second occurrence").

**Acceptance criteria:**
- [ ] [CONTRIBUTING.md](../../CONTRIBUTING.md) gains a "Tooling pitfalls" section (or extends an existing pitfalls/conventions section)
- [ ] The section names the specific failure mode, names the at-risk tools (Edit, IDE find/replace, sed), and includes a worked example drawn from the [.roughly/known-pitfalls.md:38](../../.roughly/known-pitfalls.md#L38) incident
- [ ] The section names the verification commands: `rg -nw 'old-token' <file>` and `rg -nw 'new-token' <file>` after a bulk replacement, with the expected match outputs to compare against
- [ ] Section length is 15-30 lines — long enough to be load-bearing, short enough to read in 60 seconds
- [ ] No skill, agent, or hook changes — prose-only

**Verification:**
- Reviewer reads the section cold (without prior incident context) and can describe the failure mode in their own words
- `wc -l` of the new section is in 15-30 line range

**Dependencies:** None. Slot wherever fits.

**Out of scope:**
- Code-level defense (Edit tool wrapper, lint rule, etc.) — deferred per roadmap
- Generalizing to other dual-semantic-token risks beyond this incident

---

#### E03.S6: Plan-format version field

**Maps to roadmap item:** #6

**Files touched:**
- [skills/build/SKILL.md](../../skills/build/SKILL.md) — Stage 3 plan template
- [skills/fix/SKILL.md](../../skills/fix/SKILL.md) — Stage 3 plan template
- (Possibly) [skills/build/spec-reviewer-prompt.md](../../skills/build/spec-reviewer-prompt.md) reference copy — for sync only, not behavior

**Context:**

v0.2.0 will introduce plan format v2 (complexity flag, ADR-009). v0.1.5 adds a forward-compat version line so v0.2.0's migration step can detect existing plans by version. **review-plan does NOT consume the field in v0.1.5** — that's v0.2.0's job. The field exists; nothing reads it yet.

**Acceptance criteria:**
- [ ] Plan template in [skills/build/SKILL.md](../../skills/build/SKILL.md) Stage 3 includes a `Plan-format-version: 1` line at the top (after the title, before the body)
- [ ] Plan template in [skills/fix/SKILL.md](../../skills/fix/SKILL.md) Stage 3 includes the same line
- [ ] [skills/review-plan/SKILL.md](../../skills/review-plan/SKILL.md) is unchanged — it does not validate, parse, or branch on the version field in v0.1.5
- [ ] The template change is reflected in spec-reviewer prompt reference copies (no runtime impact)
- [ ] No new ADR — ADR-009 (plan format v2) lands in v0.2.0 per roadmap
- [ ] Documentation (CHANGELOG entry under "Added") notes the field is forward-compat only

**Verification:**
- Dogfood `/roughly:build`; confirm generated plan file at `docs/plans/<name>-plan.md` has `Plan-format-version: 1` near the top
- Dogfood `/roughly:fix`; same
- `rg -n 'Plan-format-version' skills/review-plan/SKILL.md` returns no matches (review-plan doesn't consume the field)

**Dependencies:** None. Lands early so v0.2.0 work can begin parallel.

**Out of scope:**
- Any logic that reads or validates the version field
- Migration of existing plans in `docs/plans/` to add the field (those are historical artifacts)
- ADR for the field itself (folded into ADR-009 in v0.2.0)

---

### Ergonomics cluster

#### E03.S7: In-session maturity offers at Stage 1

**Maps to roadmap item:** #7

**Files touched:**
- [skills/build/SKILL.md](../../skills/build/SKILL.md) — Stage 1
- [skills/fix/SKILL.md](../../skills/fix/SKILL.md) — Stage 1

**Context:**

Maturity checks fire only at Stage 8 wrap-up today. By then the user has finished the work and may not be in a mood to take on new setup. Some checks — specifically those whose triggers are observable up-front (`investigator-v1` based on source file count, `stop-hook-v1` based on settings.json contents) — can be evaluated and offered at Stage 1 instead, when the user is fresh and uncommitted.

Checks that depend on Stage 8 state (e.g., a check that fires only after a successful run) stay at wrap-up.

**Acceptance criteria:**
- [ ] [skills/build/SKILL.md](../../skills/build/SKILL.md) Stage 1 evaluates `investigator-v1` and `stop-hook-v1` triggers and prompts up-front if conditions are met and not previously declined
- [ ] [skills/fix/SKILL.md](../../skills/fix/SKILL.md) Stage 1 does the same
- [ ] If the user accepts at Stage 1, the upgrade is applied before Stage 2 begins (recorded in `.roughly/workflow-upgrades`)
- [ ] If the user defers ("not yet"), the existing Stage 8 wrap-up offer still fires — no double-prompt, but the wrap-up serves as a second chance
- [ ] If the user declines ("never"), Stage 8 does not re-offer — same suppression as wrap-up declines
- [ ] No skill body exceeds 300 lines
- [ ] Stage 1 evaluation cost ≤ 200 tokens (read settings.json, count source files; no heavy work)

**Verification:**
- Dogfood `/roughly:build` on a project with > 50 source files and no `investigator-v1-added` record; confirm Stage 1 offer appears
- Dogfood `/roughly:build` on a project with the same conditions and an existing `investigator-v1-declined` record; confirm no Stage 1 offer
- Dogfood "not yet" response at Stage 1; confirm Stage 8 offers again at wrap-up
- Dogfood "never" response at Stage 1; confirm Stage 8 does not offer

**Dependencies:** S2 (stop-hook-v1 templating must be working); S3 (maturity-check section refactor must have landed first).

**Out of scope:**
- Moving all maturity checks to Stage 1 (some need wrap-up state)
- Adding new maturity checks
- Stage 1 evaluation of `pitfalls-organized-v1` / `test-verify-v1` (retired in S3)

---

#### E03.S8: `/roughly:help` command

**Maps to roadmap item:** #8

**Files touched:**
- `skills/help/SKILL.md` (new — 10th skill)
- [README.md](../../README.md) — mention in commands table
- [CLAUDE.md](../../CLAUDE.md) structure table — add help skill

**Context:**

There's no in-CLI overview. Users discover commands by reading SKILL.md files or the README. `/roughly:help` adds a structured at-a-glance view: list of commands grouped by purpose, current pipeline state if applicable, and current maturity check status.

Unlike pipeline/coordinator skills, this one is interactive and informational — not a gate. **`disable-model-invocation: false`**.

**Acceptance criteria:**
- [ ] New skill at `skills/help/SKILL.md` with frontmatter `name: help`, `description: <one line>`, `disable-model-invocation: false`
- [ ] Skill body under 300 lines
- [ ] Output structure: (a) commands grouped by cluster (pipeline / coordinator / utility), (b) current `.roughly/workflow-upgrades` state (which maturity checks added, which declined), (c) current pipeline state if a `docs/plans/<name>-plan.md` exists for an in-progress feature
- [ ] Skill respects pre-flight migration check (S4 conventions don't apply here — help is the recovery path itself, like upgrade)
- [ ] [README.md](../../README.md) commands table updated to include `/roughly:help`
- [ ] [CLAUDE.md](../../CLAUDE.md) structure table updated to include the new skill (count goes from 9 to 10)
- [ ] Plugin loads: `claude --plugin-dir <repo>` from a fresh project shows `/roughly:help` in autocomplete

**Verification:**
- Dogfood `/roughly:help` in a fresh project; confirm output covers all three sections cleanly
- Dogfood `/roughly:help` in this repo (mid-pipeline state); confirm pipeline-state section reports correctly
- Plugin manifest test: confirm 10 skills load

**Dependencies:** Late-cluster — depends on S2 maturity-check refactor and S3 retirements being settled, since the help output reflects those states.

**Out of scope:**
- Interactive command launch (e.g., picking a command from the help output and invoking it)
- Localization
- Help-as-a-subagent (this is a skill, not an agent)

---

#### E03.S9: Situation-specific abort prose at every pipeline failure point

**Maps to roadmap item:** #9

**Files touched:**
- [skills/build/SKILL.md](../../skills/build/SKILL.md) — every gate's abort branch
- [skills/fix/SKILL.md](../../skills/fix/SKILL.md) — same
- [skills/review-plan/SKILL.md](../../skills/review-plan/SKILL.md) — abort branches in NEEDS REVISION path
- [skills/review-epic/SKILL.md](../../skills/review-epic/SKILL.md) — abort on Risk verdict
- [skills/audit-epic/SKILL.md](../../skills/audit-epic/SKILL.md) — abort on AC failure threshold

**Context:**

Today's [skills/build/SKILL.md](../../skills/build/SKILL.md) ABORT HANDLING section (around L210) is good — it differentiates by stage (no files, plan written, implementation started). But many individual gates (review-plan NEEDS REVISION abort, Stage 6 review-fix max-cycles abort, Stage 5c question-loop max abort) emit generic "abort" messages that don't tell the user what state files are in. S9 is a sweep: every abort point produces a message naming the stage, the reason, and the state of files.

**Acceptance criteria:**
- [ ] Every abort branch in build/fix/review-plan/review-epic/audit-epic produces a message that includes: (a) what stage aborted, (b) why, (c) what files exist and in what state (plan exists/doesn't, implementation files staged/unstaged, etc.), (d) the recovery action (re-run, manually edit, escalate)
- [ ] Zero generic "Aborted." or "Pipeline aborted." messages remain — verify by `rg -n 'aborted\.?$' skills/`
- [ ] Existing ABORT HANDLING block in build/fix retained as the canonical state-of-files reference; per-gate aborts cross-link to it where appropriate
- [ ] No skill body exceeds 300 lines after the changes (this story may push close to the cap — if so, refactor to extract repeated prose into a single block referenced by gates)

**Verification:**
- `rg -n 'abort' skills/` and review every match; confirm each abort message names stage, reason, file state, recovery
- Dogfood: trigger review-plan NEEDS REVISION + abort; confirm message tells user the plan file path, that it's been written but not consumed, and how to revise
- Dogfood: trigger Stage 6 review-fix max-cycles; confirm message names which findings remain and which files are dirty

**Dependencies:** Late-cluster — sweeps across all pipeline skills, lands after pipeline-touching stories stabilize to avoid merge churn.

**Out of scope:**
- Abort handling in `/roughly:setup` and `/roughly:upgrade` (those are setup flows, not pipelines)
- Abort handling in `/roughly:help` (added in S8)
- Localization

---

#### E03.S10: Retry-loop tuning

**Maps to roadmap item:** #10

**Files touched:**
- [skills/build/SKILL.md:172](../../skills/build/SKILL.md#L172) — Stage 5c questions cap
- [skills/build/SKILL.md:176-181](../../skills/build/SKILL.md#L176-L181) — Stage 5c quality auto-fix cap
- [skills/build/SKILL.md:199](../../skills/build/SKILL.md#L199) — Stage 6 review-fix cycles cap
- [skills/fix/SKILL.md:181](../../skills/fix/SKILL.md#L181) — same questions cap
- [skills/fix/SKILL.md:185-190](../../skills/fix/SKILL.md#L185-L190) — same auto-fix cap
- [skills/fix/SKILL.md:204](../../skills/fix/SKILL.md#L204) — same review-fix cycles cap

**Context:**

v0.1.2 capped four previously-unbounded loops (per [CHANGELOG.md:82](../../CHANGELOG.md#L82)). Caps are conservative — set to "max 2" across the board with hard escalation to human on hit. v0.1.5 audits each cap individually: cheap checks may raise the cap (e.g., type-check fixes are nearly free), expensive ones may convert hard escalation to a prompt ("Continue with another auto-fix attempt? (yes / escalate)").

Each cap decision needs a before/after dogfood pass on a known case to verify behavior change is what was intended.

**Acceptance criteria:**
- [ ] All four cap sites audited; for each, the story records: (a) keep at 2, (b) raise to N, or (c) convert hard escalation to prompt — with rationale recorded inline as a one-line comment in the SKILL.md or in a new short ADR (decision pending — see [open questions](#open-questions))
- [ ] Stage 5c questions cap: decision recorded; if raised, cheap check (e.g., questions about clarification only — never about scope); if converted to prompt, prompt is one line and resolved in <100 tokens
- [ ] Stage 5c quality auto-fix cap: decision recorded; cheap auto-fix candidates (type-check, lint formatter) may raise; expensive ones (test fixes, refactors) stay at 2 or convert to prompt
- [ ] Stage 6 review-fix cycles cap: decision recorded; this is the most expensive loop — defaults to staying at 2 unless evidence supports raising
- [ ] Each adjusted cap has a before/after dogfood pass on a known case: a build/fix run that previously hit the cap, re-run after the change, with documented behavior delta
- [ ] CHANGELOG entry under "Changed" lists each cap and its v0.1.5 disposition
- [ ] No skill body exceeds 300 lines

**Verification:**
- Dogfood replay of each cap-hit case (test fixtures from S11b CI may help here)
- `rg -n 'max 2|max-2' skills/build/SKILL.md skills/fix/SKILL.md` returns expected matches given the per-cap decisions
- ADR review (if a new ADR is added)

**Dependencies:** S11 (CI) — benefits from regression coverage so cap adjustments are validated, not just edited.

**Out of scope:**
- Adding new caps to currently-uncapped loops (none exist in v0.1.5)
- Cap adjustments outside build/fix (review-epic, audit-epic, etc. — different concern)

---

### CI cluster

#### E03.S11a: Plugin self-test CI scaffolding

**Maps to roadmap item:** #11 (part 1)

**Files touched:**
- `.github/workflows/dogfood.yml` (new)
- `scripts/ci-dogfood.sh` (new)
- [CONTRIBUTING.md](../../CONTRIBUTING.md) — CI section (where to find logs, how to reproduce locally)

**Context:**

The plugin tests itself against the repo containing the plugin. Bootstrapping concerns:
- The dogfood run mutates `.roughly/`, writes plan files in `docs/plans/`, and may dirty the working tree
- A failed run leaves partial state (incomplete plan files, marker files, etc.) that can confuse subsequent CI runs and human review
- `claude` CLI usage in CI requires authentication, which is non-trivial to manage in GitHub Actions

S11a establishes the isolation contract: an ephemeral worktree, scoped teardown, and a no-pollution AC. The actual scenario logic is S11b.

**Acceptance criteria:**
- [ ] `.github/workflows/dogfood.yml` created; runs on push to main and on PR; jobs named clearly (`dogfood-build-cycle` etc.)
- [ ] `scripts/ci-dogfood.sh` created; sets up an ephemeral git worktree at `/tmp/roughly-dogfood-${SHA}` containing the plugin checkout
- [ ] Worktree teardown runs on success AND failure (`trap cleanup EXIT`); failed runs do not leave state behind
- [ ] No dogfood run mutates the source-repo working tree visible to subsequent commits — verify by checking `git status --porcelain` is unchanged before and after a CI run
- [ ] `claude` CLI authentication is handled via a documented secret (e.g., `ANTHROPIC_API_KEY`); CONTRIBUTING.md explains how to set it
- [ ] `scripts/ci-dogfood.sh` is runnable locally with the same env vars, producing the same behavior
- [ ] CONTRIBUTING.md gains a CI section: where workflow logs live, how to reproduce a failure locally, what's in scope vs out of scope for CI

**Verification:**
- Push a no-op branch to a fork; confirm dogfood.yml fires, completes (with a stub scenario), and tears down cleanly
- Locally invoke `bash scripts/ci-dogfood.sh`; confirm same behavior, no source-tree pollution (`git diff --quiet && git status --porcelain | grep -q .` returns no surprises)
- Inspect the worktree path during a run; confirm it's isolated from the source checkout

**Dependencies:** S1 — without plan-mode auto-detect, the dogfood script can't safely invoke `/roughly:build` from a CI session that may auto-engage plan mode.

**Out of scope:**
- The actual build-cycle scenario (S11b)
- Coverage of `/roughly:fix`, `/roughly:setup`, `/roughly:upgrade` (S11b expands; for v0.1.5 happy-path-only)
- Caching node_modules / Claude state between runs (correctness first, perf later)

---

#### E03.S11b: Scripted dogfood happy-path build cycle

**Maps to roadmap item:** #11 (part 2)

**Files touched:**
- `scripts/ci-dogfood.sh` (extend)
- `tests/fixtures/<name>/` (new — fixture repo for the scenario)

**Context:**

S11a establishes isolation; S11b drives the actual scenario. The hard problem: a build cycle includes Stage 4 plan review, which today requires human input on PASS / NEEDS REVISION + override. CI must drive this without a human.

Options for the format are documented in [open questions](#open-questions); the story commits to a happy-path scenario only.

**Acceptance criteria:**
- [ ] `tests/fixtures/<name>/` contains a minimal repo (CLAUDE.md, one source file, a trivial test) that exercises the build pipeline end-to-end
- [ ] `scripts/ci-dogfood.sh` invokes `/roughly:build` against the fixture for a small, deterministic feature ("add a constant" or similar)
- [ ] The scenario succeeds: plan written, review-plan returns PASS, implementation runs, verify-all passes, wrap-up records workflow upgrades, no abort
- [ ] CI fails loudly if any stage produces unexpected output (silent failure mode protection)
- [ ] CI fails loudly if Stage 4 review-plan is not invoked (plan-mode hijack regression protection — relies on S1)
- [ ] The scenario completes in under 5 minutes wall-clock on GitHub Actions standard runners
- [ ] Failure logs include enough context (stage reached, last 50 lines of output) to diagnose without re-running locally

**Verification:**
- Push a change that breaks Stage 4 dispatch; confirm CI fails with a clear message
- Push a change that breaks plan-mode detection (S1 regression); confirm CI fails with a clear message
- Push a clean change; confirm CI passes in <5 min

**Dependencies:** S11a (scaffolding), S1 (plan-mode detection), S6 (plan format version field — shouldn't break the scenario but worth verifying), and ideally S9 (clearer abort prose helps CI failure diagnosis)

**Out of scope:**
- `/roughly:fix` scenario (next release)
- `/roughly:setup` scenario (next release)
- Negative-path scenarios (review-plan NEEDS REVISION, Stage 6 max cycles, etc.)
- Performance benchmarking
- Cross-platform CI (Linux only)

---

### Docs cluster

> **Note:** Per [docs/planning/README.md:84](../../docs/planning/README.md#L84), the `roughly.dev` site is "out of repo scope; tracked separately." The PM prompt asserts docs are part of v0.1.5 DoD. Resolution is deferred to [open questions](#open-questions); for now, S12a/b assume docs source content lands in this repo under `docs/site/` and is published to roughly.dev as a separate manual step. If the user prefers stories that work directly against a separate roughly.dev repo, S12a/b restructure accordingly.

#### E03.S12a: roughly.dev landing + setup walkthrough

**Maps to roadmap item:** #12 (part 1)

**Files touched:**
- `docs/site/index.md` (new — landing page source)
- `docs/site/setup.md` (new — setup walkthrough source)

**Context:**

The roadmap floor for v0.1.5 docs is four pages: landing, pipeline overview, commands reference, setup walkthrough. S12a covers landing + setup walkthrough; S12b covers the other two. Splitting reduces final-week landmine risk.

**Acceptance criteria:**
- [ ] `docs/site/index.md` (landing): one-paragraph thesis (paraphrased from ROADMAP), three-bullet "what Roughly does" summary, install command, link to setup walkthrough; budget: 80-150 lines including markdown
- [ ] `docs/site/setup.md` (setup walkthrough): `/roughly:setup` step-by-step for a single-project install, including the maturity check decision tree (which checks fire when, what they offer); budget: 120-200 lines including code blocks
- [ ] Both pages cross-link to commands reference (S12b) with placeholder anchors that resolve once S12b lands
- [ ] No prose contradicts the in-repo SKILL.md files (verify by reviewer cross-check)
- [ ] No marketing voice ("industry-leading", "revolutionary", etc.) — engineer-to-engineer tone matching the roadmap

**Verification:**
- Reviewer reads both pages cold; can describe what Roughly does and run `/roughly:setup` correctly without reading SKILL.md
- `wc -l` on each file is within budget

**Dependencies:** None blocking; ladders early in the release.

**Out of scope:**
- Pipeline overview (S12b)
- Commands reference (S12b)
- Site framework / build tooling for roughly.dev (separate repo)
- Visual design beyond plain markdown

---

#### E03.S12b: roughly.dev pipeline overview + commands reference

**Maps to roadmap item:** #12 (part 2)

**Files touched:**
- `docs/site/pipeline.md` (new — pipeline overview)
- `docs/site/commands.md` (new — commands reference)

**Context:**

Pipeline overview = the 8 build stages + abort handling + maturity checks, written for a stranger who has not read SKILL.md. Commands reference = the 10 commands (post-S8) with one-line summary, when to use, what they produce.

**Acceptance criteria:**
- [ ] `docs/site/pipeline.md`: covers all 8 build stages with one paragraph each, plus abort handling and maturity checks; budget: 200-350 lines
- [ ] `docs/site/commands.md`: lists all 10 commands (`/roughly:build`, `/roughly:fix`, `/roughly:review`, `/roughly:review-plan`, `/roughly:review-epic`, `/roughly:audit-epic`, `/roughly:verify-all`, `/roughly:setup`, `/roughly:upgrade`, `/roughly:help`); each entry has: one-line summary, when to use, what it produces, link to relevant SKILL.md anchor; budget: 150-250 lines
- [ ] Cross-links from S12a's landing/setup pages resolve correctly
- [ ] No prose contradicts the in-repo SKILL.md files (verify by reviewer cross-check)
- [ ] Engineer-to-engineer tone, no marketing voice

**Verification:**
- Reviewer reads both pages cold; can describe the build pipeline accurately and pick the right command for a given situation
- `wc -l` on each file is within budget
- All cross-references resolve (no broken anchor links)

**Dependencies:** S8 (must include `/roughly:help` in commands reference — sequence S12b after S8 lands).

**Out of scope:**
- ADR summaries (deferred — link to ADRs is sufficient)
- Tutorial content beyond setup
- Migration guides from other workflows

---

## Open questions

These are surfaced in story bodies but consolidated here for the implementer's convenience.

1. **CI scripted build cycle format (S11b).** Options:
   - **(a) Heredoc-fed answers** — pipe canned PASS/override responses via stdin
   - **(b) Override-token env var** — set `ROUGHLY_CI_AUTO_PASS=true` in the build skill's review-plan dispatch
   - **(c) Mock-mode flag in build skill** — `/roughly:build --ci` shortcuts review-plan with a synthetic PASS verdict
   - Each has trade-offs: (a) most realistic but brittle to skill prompt changes; (b) clean but requires skill modification; (c) cleanest but skill modification is more invasive. Decision needed before S11b implementation.

2. **Maturity check replacement coverage (S3).** Doc-writer fires on pipeline-driven writes only. Manual edits to `.roughly/known-pitfalls.md` (user opening it in their editor) won't trigger the organize-suggestion. Acceptable coverage loss for v0.1.5, or push triggers into `.claude/hooks/verify-all.sh` (Stop hook from S2) so manual edits are caught at next session boundary?

3. **Retry-loop per-cap decisions (S10).** Each of the four caps may stay, raise, or convert to prompt. Decisions need to be made before S10 lands; recommend the implementer prepare a per-cap rationale doc and review with the user before editing.

4. **roughly.dev source location (S12).** [docs/planning/README.md:84](../../docs/planning/README.md#L84) says the site is "out of repo scope." The PM prompt says docs are part of v0.1.5 DoD. Reconciliation options:
   - **(a) Source content lives in this repo at `docs/site/`**, manual publish to roughly.dev separate
   - **(b) Stories work directly in a separate roughly.dev repo** (cross-repo coordination needed)
   - **(c) v0.1.5 DoD redefines** docs source location and the planning-README note is updated
   - Decision needed before S12a starts.

---

## v0.1.6 candidates

Items surfaced during epic writing that are clearly related to v0.1.5 work but explicitly out of frozen scope:

- **Marker-aware resume improvements in [skills/upgrade/SKILL.md](../../skills/upgrade/SKILL.md)** — surfaced while scoping S4. Today's `/roughly:upgrade` migration logic handles `.ruckus/.migration-in-progress` markers, but there's room to make resume reporting cleaner (which steps already ran, which still need to). Not blocking v0.1.5 since the marker mechanism is correct as-is.
- **Expanded plan-mode signals if S0 spike reveals additional gaps** — if S0 finds the preamble-only mechanism leaves a known hole, additional defense (Stop-hook check, etc.) is a v0.1.6 candidate rather than v0.1.5 scope expansion.
- **Per-field maturity-check organization beyond v1 IDs** — S3's retirement raises the question of whether the existing v1 IDs are themselves the right grain. Deferred.
- **Manual-edit detection for `.roughly/known-pitfalls.md`** (relates to open question 2) — pushing organize-suggestion logic into the Stop hook so manual edits are caught.
- **CI coverage for `/roughly:fix`, `/roughly:setup`, `/roughly:upgrade`** (S11b is happy-path build only). Per-command CI scenarios.
- **Negative-path CI scenarios** (review-plan NEEDS REVISION, Stage 6 max cycles, abort recovery).

---

## Sequencing

Order is by dependency, not roadmap item number.

| # | Story | Why this position |
|---|---|---|
| 1 | **E03.S0** (plan-mode spike) | ½-day investigation; gates S1 |
| 2 | **E03.S1** (plan-mode auto-detect/exit) | Highest-value item; unblocks safe CI dogfood runs |
| 3 | **E03.S11a** (CI scaffolding) | Lands early so subsequent stories ship with regression coverage |
| 4 | **E03.S6** (plan-format version field) | Additive, low-risk; lands next so v0.2.0 work can begin parallel |
| 5 | **E03.S5** (CONTRIBUTING prose) | Independent, prose-only; slot anywhere |
| 6 | **E03.S4** (pre-flight in audit-epic + verify-all) | Independent of pipeline changes |
| 7 | **E03.S3** (retire test-verify-v1 / pitfalls-organized-v1) | Folds triggers into doc-writer; doesn't break anything |
| 8 | **E03.S2** (stop-hook-v1 templating) | After S3 to avoid double-touching maturity check section |
| 9 | **E03.S7** (in-session maturity offers at Stage 1) | Depends on S2 + S3 maturity refactor |
| 10 | **E03.S12a** (docs landing + setup) | Ladders mid-release rather than batch-landing at the end |
| 11 | **E03.S9** (situation-specific abort prose) | Sweep across pipeline skills; lands late to avoid merge churn |
| 12 | **E03.S10** (retry-loop tuning) | Late; benefits from CI regression coverage from S11 |
| 13 | **E03.S11b** (full dogfood scenario) | After pipeline-touching stories stabilize |
| 14 | **E03.S8** (`/roughly:help` command) | Late; documents the final shape of the release |
| 15 | **E03.S12b** (docs pipeline + commands) | After S8 so commands reference includes `/roughly:help` |

---

## Definition of done

- All 15 stories merged
- v0.1.5 tag pushed
- CHANGELOG entry covers Added / Changed / Fixed / Notes for each story
- ROADMAP.md updated to reflect v0.1.5 shipped + v0.1.6 candidates surfaced
- CI dogfood run passing on main
- roughly.dev landing, setup, pipeline, commands pages live (location per open question 4)
