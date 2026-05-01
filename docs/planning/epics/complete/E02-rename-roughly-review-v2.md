# E02 Rename Epic — Pre-Implementation Review (v2)

**Reviewer:** Ruckus epic-reviewer (Opus)
**Date:** 2026-04-28
**Source epic:** `docs/planning/epics/E02-rename-roughly.md` (v2 — 729 lines)
**Prior review:** `docs/planning/epics/E02-rename-roughly-review.md` (v1 review)
**Verdict:** Ready (with three narrow corrections; none block S2.1 kickoff)

---

## Summary

v2 closes 9 of 10 v1 recommendations cleanly. The 10-point migration spec is internally consistent, the marker-file partial-failure mechanism works correctly under both the half-failed-move and abort-then-resume scenarios when walked through, and the line counts (build=294, fix=299, upgrade=125) match the epic's claims to the byte. The agent-preamble.md L3–L13 structure verified exactly as cited. Two narrow line-citation corrections in S2.7's allow-list are needed (capital-R `Ruckus` prose lines for ADR-004, ADR-005, and ADR-006 are partially miscited), and the README migration-subsection placement guidance has a ~70-line insertion zone that should be tightened to a single anchor. Neither blocks implementation kickoff — they can be fixed in S2.5/S2.7 directly without revising upstream stories.

---

## v1 finding closures (10/10 addressed; 1 closed-with-issue)

1. **S2.3 line citation (L6–L8 → L3–L13).** Closed correctly. Verified actual file: L3 `<!--`, L4 prose, L5–L8 manual-sync targets list (5 inliners + 2 inlined templates per L7–L8 wording), L9–L13 exceptions paragraph (static-analysis L9–L11, doc-writer L12), L13 `-->`. Epic L257, L279–L289 cite this accurately. Render-validation AC at epic L319 added correctly.

2. **S2.2 partial-failure idempotency.** Closed correctly. The marker mechanism at points 2–3, 5–6, 9–10 works under the spelled-out half-failed scenario: marker written before any move; if `git mv known-pitfalls.md` succeeds and `git mv workflow-upgrades` fails, marker remains; on re-run, point 3's "marker present → resume from step 5" branch fires; step 5's "if source does not exist (already moved), skip silently" handles known-pitfalls.md; step 5 then moves workflow-upgrades; step 6 idempotency on the version-line-identifier rewrite handles the case where the already-moved file may already have been rewritten. Resume path is sound.

3. **S2.2 abort behavior.** Closed correctly **but with one UX subtlety worth flagging** (see NEW issues #2 below). The epic L172 explicitly documents abort as "stop the upgrade entirely" and "Do NOT remove the marker on abort; the next run will resume." However, the marker is only written at point 2 if the user is past the conflict prompt — abort fires from inside point 3 BEFORE point 2 has run on the conflict-with-no-marker path. So on abort+rerun, the conflict prompt fires again. This is correct re-entry, not "resume," and the L172 wording "the next run will resume" is technically wrong for this branch.

4. **S2.7 allow-list for capital-R `Ruckus` ADR prose.** Closed but **with line-citation drift** (see NEW issues #1). The allow-list bullet was added (epic L565) but several cited line numbers are wrong — see Line-citation spot-check below.

5. **README + CHANGELOG migration discoverability.** Closed correctly in spec content, **but README placement is loose** (see NEW issues #3). README subsection content at L434–L448 is well-formed; CHANGELOG `### Migration` at L604–L608 mirrors the README content faithfully (verified line-by-line: install command, run upgrade, optional uninstall — wording aligns).

6. **S2.2 boilerplate-rewrite warning.** Closed correctly. Anchored regex `^Pitfalls.*\/ruckus:build.*\/ruckus:fix.*$` (epic L180) matches the template-installed line at known-pitfalls.md.template L6 (verified). It will not match a customized line, in which case the silent-skip + warning is the documented behavior. The "false alarm" warning on resume cases (when the line already reads `/roughly:*`) is correctly acknowledged as harmless.

7. **S2.7 v0.1.2 step regression test (5b).** Closed correctly. Test scaffolding at epic L582 specifies `docs/claude/known-pitfalls.md` AND `docs/claude/.workflow-upgrades` — verified against upgrade/SKILL.md L19 which sources from `docs/claude/.workflow-upgrades` (with leading dot), correctly modeling the pre-v0.1.2 user state.

8. **S2.5 ADR footnote placement.** Closed correctly. Epic L423–L426 spells out: ADR-005/006 append below existing v0.1.2 footnote with a single blank line separator; ADR-004/008 get the footnote as the new last line. Verified ADR-005 L52 and ADR-006 L47 contain the v0.1.2 footnote already.

9. **Tightened S2.2 grep ACs.** Closed correctly. Epic L229–L231 reframes greps as "verified by running the grep then auditing each match against the allow-list" — not pure zero-match. Allow-list explicitly enumerates v0.1.2 and v0.1.4 migration prompt regions.

10. **S2.3 render-validation AC.** Closed correctly. AC at epic L319 explicitly requires "When agent-preamble.md is rendered, the visible output is L1 heading + L15–L17 instruction body only" with falsification criterion "if any prose from L4–L12 appears in the rendered output, the `<!--` or `-->` delimiter has been broken."

---

## NEW issues introduced by v2

### #1 S2.7 allow-list line citations for ADR `Ruckus` prose are partially wrong (HIGHEST PRIORITY)

Epic L565: `"Capital-R Ruckus prose in ADR body text (ADR-004 L11, L17, L27; ADR-005 L11, L17, L35; ADR-006 L11, L17; ADR-008 L11)"`. Verified each cited line:

- **ADR-004 L11/L17/L27**: NONE of these are capital-R `Ruckus` prose. L11 contains `/ruckus:build-ui` and `/ruckus:build` (lowercase `/ruckus:` namespace tokens, not the capital-R word); L17 contains `/ruckus:build` only; L27 contains `/ruckus:build` and `/ruckus:build-ui` only. ADR-004 has zero capital-R `Ruckus` prose anywhere — verified by reading the full file.
- **ADR-005 L11**: `"Ruckus adapts its behavior..."` — correct, capital-R prose. **L17**: `"every /ruckus:build and /ruckus:fix invocation"` — lowercase namespace, NOT capital-R prose. **L35**: `"explicitly run /ruckus:upgrade"` — lowercase namespace, NOT capital-R prose.
- **ADR-006 L11**: `"Ruckus is a plugin..."` — correct, capital-R prose. **L17**: contains `.ruckus/known-pitfalls.md` (lowercase path), NOT capital-R prose.
- **ADR-008 L11**: `"Ruckus dispatches multiple subagents..."` — correct.

The drift means the S2.7 allow-list bullet conflates two distinct allow-list categories (capital-R `Ruckus` prose vs. lowercase `/ruckus:*` namespace and `.ruckus/` path tokens in ADR body text). Both ARE allow-listed by the broader bullets at L566 (`Inline /ruckus:* references in ADR-004 and ADR-005 body text`), but ADR-006's lowercase `.ruckus/` path at L17 is not explicitly covered, and the audit grep `rg -i ruckus` will flag it.

**Fix:** replace L565–L566 with three separate, accurate bullets:
- `Capital-R Ruckus prose in ADR body text: ADR-005 L11, ADR-006 L11, ADR-008 L11 (only these three lines).`
- `Inline /ruckus:* namespace tokens in ADR body text: ADR-004 L11/L17/L27, ADR-005 L17/L35.`
- `Lowercase .ruckus/ path references in ADR body text: ADR-005 L17, ADR-006 L17.`

Story-level: **S2.7**.

### #2 S2.2 marker UX has misleading note

Epic L172: `"Do NOT remove the marker on abort; the next run will resume."` This is incorrect for the conflict-with-no-marker abort path. On that path, point 2 (marker write) runs only AFTER the conflict prompt is resolved with a non-abort answer. So if the user picks `abort` at the no-marker conflict prompt, no marker was ever written. On re-run, marker is still absent → conflict prompt fires AGAIN, not "resume." This is correct UX behavior (the user has not made a resolution decision yet), but the epic text "the next run will resume" mischaracterizes it.

**Fix:** change L172 to `"Do NOT remove the marker on abort. (For the conflict-prompt path: marker was never written; re-running re-prompts. For the partial-failure path with a marker present: re-running resumes from step 5.)"`.

Story-level: **S2.2**.

### #3 S2.5 README placement guidance is loose

Epic L432 says `(placement: after the install/quick-start sections, before the Skills Reference)`. Verified README structure: `## Installation` L5, `## Quick Start` L14, `## How It Works` L57, `## Understanding Gates` L71, `## Choose Your Workflow` L92, `## Skills Reference` L105. The cited zone is L31–L105 — about 70 lines wide with three intermediate sections (How It Works / Understanding Gates / Choose Your Workflow) where the migration subsection could land.

**Fix:** anchor to a single insertion point — recommend `"directly after the Your first feature subsection (currently ending at L55) and immediately before How It Works (L57)"` so the migration block sits naturally in the install-and-onboard region.

Story-level: **S2.5**.

### #4 S2.2 marker write may fail silently on read-only `.ruckus/`

Edge case: if `.ruckus/` has been chmod'd read-only by the user (rare but real on locked-down dev machines), point 2's marker write will throw before any move is attempted. The 10-point spec does not handle this — there is no "if marker write fails" branch. **Severity: low** (user can chmod and re-run; failure is loud, not silent).

**Fix:** add to point 2: `"If the marker write fails (read-only filesystem, permissions), abort with: 'Cannot write marker to .ruckus/.migration-in-progress — check directory permissions.' No partial state is created."`

Story-level: **S2.2**.

### #5 CHANGELOG `### Migration` is non-standard

Per Keep-a-Changelog convention, sections are Added/Changed/Deprecated/Removed/Fixed/Security. `### Migration` is custom. Acceptable for human-readable changelog but may surprise tooling. **Severity: very low** — flag only, not a blocker.

Story-level: **S2.7**.

### #6 S2.7 AC L621 wording (stylistic)

`"S2.6's dogfood .ruckus/ → .roughly/ rename has merged before this story's audit runs"` mixes a process-step with a state assertion. The parenthetical fixes it: `(verified: ls -la shows .roughly/ and no .ruckus/ at repo root)`. The AC works as written because of the parenthetical, but stylistically the "has merged before this story's audit runs" half is unobservable post-hoc. **Severity: stylistic** — leave as-is.

### #7 S2.7 step 8 user-extras prompt → marker cleanup ordering

Epic L184–L186: step 8 prompts; if user says `leave`, point 9 says "do NOT remove `.ruckus/`" but does NOT say what to do with `.ruckus/.migration-in-progress`. Should the marker be removed? Logical answer: yes — migration is "complete enough" (the user explicitly elected to handle their extras manually), so the marker should not trigger a partial-migration resume on the next run.

**Fix:** add to point 9: `"Remove the marker file regardless of whether .ruckus/ is removed. The marker indicates 'migration in progress'; once the standard moves are done and the user has dispositioned extras, the migration is complete."`

Story-level: **S2.2**.

---

## Line-citation spot-check results

| Citation | Status |
|---|---|
| agent-preamble.md L3 `<!--`, L13 `-->` | Verified |
| agent-preamble.md L5–L8 manual-sync target list | Verified |
| agent-preamble.md L9–L13 exceptions paragraph | Verified |
| skills/build/SKILL.md = 294 lines | Verified |
| skills/fix/SKILL.md = 299 lines | Verified |
| skills/upgrade/SKILL.md = 125 lines | Verified |
| ADR-004 L11/L17/L27 (claimed: capital-R Ruckus prose) | **DRIFT — these are lowercase /ruckus: namespace tokens, NOT capital-R prose. ADR-004 has no capital-R Ruckus prose.** |
| ADR-005 L11 (capital-R Ruckus prose) | Verified |
| ADR-005 L17/L35 (claimed: capital-R Ruckus prose) | **DRIFT — these are lowercase /ruckus: namespace tokens** |
| ADR-005 L52 (existing v0.1.2 footnote) | Verified (file is 52 lines total, footnote is the last line) |
| ADR-006 L11 (capital-R Ruckus prose) | Verified |
| ADR-006 L17 (claimed: capital-R Ruckus prose) | **DRIFT — this line contains lowercase .ruckus/known-pitfalls.md path, NOT capital-R prose** |
| ADR-006 L47 (existing v0.1.2 footnote) | Verified (file is 47 lines total, footnote is the last line) |
| ADR-008 L11 (capital-R Ruckus prose) | Verified |
| known-pitfalls.md.template L6 starts with `Pitfalls` | Verified — regex anchor `^Pitfalls` matches |
| `.ruckus/workflow-upgrades` (no leading dot, dogfood) | Verified |
| `docs/claude/.workflow-upgrades` (with leading dot, legacy v0.1.0/v0.1.1 source format per upgrade/SKILL.md L19) | Verified — S2.7 step 5b test scaffolding correctly uses the dotted name |

---

## Top-3 concerns for v2

1. **S2.7 allow-list line-citation drift (NEW issue #1)** — three of the cited "capital-R Ruckus prose" lines are actually lowercase namespace tokens. Without a fix, the implementer running the static residue audit will either flag legitimate matches as failures and not understand why, or silently broaden the allow-list and lose audit precision. Single highest-priority fix; affects S2.7 only; takes one targeted edit to L565–L566.

2. **S2.2 marker abort wording (NEW issue #2)** — `"the next run will resume"` is wrong for the no-marker abort branch. The behavior is correct; the explanation is misleading. Implementer may write code that expects to skip the conflict prompt on re-run after abort, which would violate the documented UX (re-prompt on every conflict-no-marker run until user picks a non-abort option).

3. **README migration subsection placement zone is too wide (NEW issue #3)** — 70-line zone with three intermediate sections. The implementer needs a single anchor or the placement decision becomes a judgment call that the PM may not endorse on review.

None of these block kickoff of S2.1. All three can be fixed in-flight when S2.2/S2.5/S2.7 are scheduled. **v2 is implementable as-is** — the three corrections above improve precision but the epic does not contain any architectural defect or under-specified core path.

---

## Files referenced

- /Users/nickkirkes/rowdycloud/code/ruckus/docs/planning/epics/E02-rename-roughly.md
- /Users/nickkirkes/rowdycloud/code/ruckus/docs/planning/epics/E02-rename-roughly-review.md
- /Users/nickkirkes/rowdycloud/code/ruckus/agents/agent-preamble.md
- /Users/nickkirkes/rowdycloud/code/ruckus/skills/build/SKILL.md
- /Users/nickkirkes/rowdycloud/code/ruckus/skills/fix/SKILL.md
- /Users/nickkirkes/rowdycloud/code/ruckus/skills/upgrade/SKILL.md
- /Users/nickkirkes/rowdycloud/code/ruckus/skills/setup/templates/known-pitfalls.md.template
- /Users/nickkirkes/rowdycloud/code/ruckus/docs/adrs/ADR-004-ui-conditional-not-forked.md
- /Users/nickkirkes/rowdycloud/code/ruckus/docs/adrs/ADR-005-versioned-maturity-checks.md
- /Users/nickkirkes/rowdycloud/code/ruckus/docs/adrs/ADR-006-runtime-context-not-baked.md
- /Users/nickkirkes/rowdycloud/code/ruckus/docs/adrs/ADR-008-opus-for-epic-reviewer-only.md
- /Users/nickkirkes/rowdycloud/code/ruckus/README.md
- /Users/nickkirkes/rowdycloud/code/ruckus/.ruckus/workflow-upgrades
- /Users/nickkirkes/rowdycloud/code/ruckus/.ruckus/known-pitfalls.md
