# E02 Rename Epic — Pre-Implementation Review

**Reviewer:** Ruckus epic-reviewer (Opus)
**Date:** 2026-04-28
**Source epic:** `docs/planning/epics/E02-rename-roughly.md`
**Verdict:** Needs Revision

---

## Summary

This is a well-prepared epic — discovery is thorough, ADR fidelity is principled, and the parallelization plan is correct. The 12-agent validation pass shows. However, several spot-checked line citations are slightly off, a couple of grep-based ACs will over-match in ways that produce false-positive failures, and the new 8-point v0.1.4 migration step in S2.2 has under-specified edge cases (most importantly: idempotency on partial failure, and what "Remove empty `.ruckus/`" means when `git rm -r` semantics aren't called out). None are showstoppers, but they will cost an implementation cycle if shipped as-is.

---

## Top-3 Highest-Confidence Concerns

1. **Migration discoverability gap (Risks).** Existing v0.1.3 users have `/ruckus:upgrade`, not `/roughly:upgrade`. The epic does not document a clear path for them to discover and execute the migration. This is the most user-visible failure mode of a "hard cut, no aliases" rename.

2. **S2.2 partial-failure idempotency (Technical Accuracy / Risks).** Spec point 8 only handles the "fully migrated" re-run case. A mid-migration crash leaves `.ruckus/` and `.roughly/` both populated, and the conflict prompt at point 2 misclassifies this as user-data conflict when it's actually a half-completed migration. Needs explicit handling.

3. **S2.7 allow-list incompleteness for capital-R `Ruckus` in ADR body text (AC Quality / Dependencies).** The audit will trip on legitimate historical prose in ADR-004/005/006/008. Without an allow-list bullet, the implementer either fails the AC or silently rewrites ADR body text — violating the "ADR immutability except footnote" principle the epic is built on.

---

## By Dimension

### Technical Accuracy

**S2.3 — agent-preamble.md L6–L8 claim is wrong.** Epic says the "header comment listing manual-sync targets" is at L6–L8. Actual file: the HTML comment block runs L3–L13 (`<!--` at L3, closing `-->` at L13, with the manual-sync target list spanning L5–L8 and the exceptions list L9–L13). The L6–L8 spot-check is meaningless — those three lines are a fragment inside a larger comment. ACs in S2.3 that say "L6–L8 header comment includes doc-writer" will be ambiguous to the implementer. Recommend changing the citation to "the HTML comment at L3–L13" and noting the target sub-region.

**S2.5 — ADR-008 has no `/ruckus:*` body text and no L-cited references in the epic.** Verified: `grep -n "/ruckus:" docs/adrs/ADR-008*.md` returns zero. ADR-008's only "ruckus" reference is L11 ("Ruckus dispatches multiple subagents..."), capital-R prose. The story says ADR body text remains unchanged, which leaves L11 untouched — consistent with the "historical decision text" rationale, but it should be made explicit so the S2.7 `rg -i ruckus` audit doesn't flag it without a clear allow-list entry.

**S2.2 — `rg -n '\bRuckus\b' skills/` AC will misfire.** The AC says "Zero capital-R `Ruckus` prose in skills". But `\bRuckus\b` will also match inside the new v0.1.4 migration prompt text in upgrade/SKILL.md, which the spec at L156 says reads `"Roughly renamed `docs/claude/` to `.roughly/`"`. If the implementer follows S2.5's "preserve historical decision text" principle and puts a sentence like "renamed from Ruckus to Roughly" in the migration prompt, the grep AC fails. Either tighten to "Zero `Ruckus` outside the v0.1.2 and v0.1.4 migration prompt text" or add an allow-list bullet.

**S2.2 spec point 5b — boilerplate regex is fragile.** The spec at epic L170 says: rewrite the boilerplate header line "matching the template-installed pattern `Updated by .*\/ruckus:build.*\/ruckus:fix`". This regex fails on a user who has reordered the commands ("Updated by `/ruckus:fix` and `/ruckus:build`") or who localized the wording. The spec correctly says "If the boilerplate line has been modified or removed by the user, skip this rewrite silently" — so the failure mode is silent skip, which is safe. But the implementer should be told to anchor the match to the start of the line and, on a non-match, log a one-line warning to the upgrade summary so the user knows their custom boilerplate was preserved verbatim and may still contain stale `/ruckus:*` tokens.

**S2.2 spec point 4 — `git mv` vs `mv` is correctly hedged.** Verified the dogfood repo IS a git repo, but the user's project may not be. The "(Plain `mv` is acceptable if not in a git repo.)" parenthetical is correct, but the implementer should be told to detect with `git rev-parse --git-dir 2>/dev/null` rather than try `git mv` first and fall back. Otherwise the failed `git mv` may emit confusing output to the user.

**S2.2 spec point 8 — idempotency check is insufficient for partial failure.** The spec says "Re-running the upgrade after a successful v0.1.4 migration finds no `.ruckus/` and skips this step." But what about partial failure mid-migration? Concrete: `git mv .ruckus/known-pitfalls.md` succeeds, `git mv .ruckus/workflow-upgrades` fails (file locked, permission, etc.), step exits with error. Re-run: `.ruckus/` still exists (contains workflow-upgrades) AND `.roughly/` exists (contains known-pitfalls.md). The conflict-check at point 2 fires — but the prompt "Both .ruckus/ and .roughly/ exist. Show diff and ask which to keep?" is misleading because this is NOT a user-data conflict, it's a half-completed migration. Recommend: detect "partial migration" via timestamp heuristic or write a `.ruckus/.migration-in-progress` marker before starting and remove on success.

**S2.2 conflict-check prompt is a 4-option prompt mislabeled as 3-option.** Epic L165: `"Both .ruckus/ and .roughly/ exist. Show diff and ask which to keep? (diff / keep .roughly / keep .ruckus / abort)"`. The behavior of `abort` is undocumented. The v0.1.2 step uses abort to stop the entire upgrade. Recommend: "On `abort`: stop the upgrade entirely with the same message pattern as the v0.1.2 step."

**S2.2 spec point 6 — root CLAUDE.md update silently skips on user customization.** Reasonable, but if the user has a CUSTOMIZED reference like `` `.ruckus/known-pitfalls.md` `` (escaped backticks) or `<!-- ruckus -->` (HTML comment), point 6's literal substring replace MAY produce edge-case mojibake. The AC should require literal-substring match (not regex) with explicit before/after counts displayed in the upgrade summary.

**S2.7 static residue audit allow-list — incomplete.** The audit must be run AFTER all stories merge AND after the `.ruckus/` → `.roughly/` rename in S2.6, because `rg -i ruckus` against the old directory name will produce false residue.

### Best Practices

- ADR-driven design preserved — footnote-only convention parallels v0.1.2's ADR-005/006 footnotes (verified at L52 and L47 respectively). Strong.
- ADR-006 runtime context loading preserved — paths change, semantics don't.
- ADR-003 manual-sync pattern correctly identified; S2.3 explicitly executes the manual sync. Good.
- Per CLAUDE.md "Maturity check IDs must be versioned": the epic correctly notes no version bump is needed because none of the existing IDs embed "ruckus" (verified — `investigator-v1`, `pitfalls-organized-v1`).
- **Concern:** Per CLAUDE.md "Skill bodies must stay under 300 lines": build = 294, fix = 299. The fix/SKILL.md "≤ 300 lines after edits" AC has a 1-line budget. If the implementer accidentally adds a single newline, the AC fails. Recommend either bumping the AC to ≤ 305 lines (with documented rationale) or explicitly noting "edits must be in-place character substitutions only."

### Risks

- **Existing v0.1.3 users have `/ruckus:upgrade`, not `/roughly:upgrade`.** Highest-impact discoverability gap. After the GH repo rename + new install, users will install `roughly` but their old `ruckus` plugin cache may still resolve `/ruckus:upgrade`. Two plugins installed, two slash commands, both functional — and only `/roughly:upgrade` runs the migration. Recommend: S2.7 step or new S2.8 to add a CHANGELOG note AND a README "Migrating from ruckus" subsection with explicit two-step instructions: (1) install the new plugin under name `roughly`, (2) run `/roughly:upgrade` from each project, (3) optionally `/plugin uninstall ruckus`.

- **User has additional files in `.ruckus/` beyond the two known files.** S2.2 spec point 7: "If non-empty after the migration (user added files), leave it and warn." Stronger: surface this as a Phase 2 prompt, not a trailing warning, so the user can decide whether to also `git mv` those files into `.roughly/`.

- **Mixed-content lines (build L250, fix L255, setup L40) under serial S2.1 → S2.2.** If S2.1's edit accidentally rewrites the prose `Ruckus` because the implementer did a wholesale line replacement, S2.2 has no-op. Recommend S2.1's task description explicitly say "edit ONLY the `/ruckus:*` token; leave `Ruckus` prose for S2.2" and S2.2 say "if S2.1 already replaced the prose, no-op."

- **Tag v0.1.4 created locally before GH repo rename.** Risk: tag created on commit X, further commits land between tag and push (e.g., a hotfix). Recommend explicitly documenting "no commits land between tag-create and tag-push." Confirmed: nothing in plugin.json or marketplace.json embeds the GH repo URL, so the tag content is repo-rename-agnostic.

- **`docs/planning/**` is gitignored.** E02-rename-roughly.md itself is not version-controlled. Validation (12 agents, this review) won't be visible to anyone reading the repo history. Implementation handoff should snapshot to a non-ignored location post-merge.

### Overengineering

- **The 8-point migration spec is appropriate, not overengineered.** Each point handles a real edge case discovered during validation. The detail is justified by the "hard cut, no aliases" decision.

- **ADR footnote-only approach is principled, not pedantic.** Continuing the v0.1.2 convention keeps the ADR archive trustworthy as a historical record.

- **S2.5's preservation of inline `/ruckus:*` in ADR body text — principled.** ADRs document "what we decided at time T." Rewriting the namespace tokens in decision text would falsify what was decided.

- **Minor:** S2.3's preamble drift sync includes doc-writer despite its different pattern. A simpler approach: keep the existing 5-agent preamble-sync list, add a separate parallel comment line "Path-string sync also required for doc-writer L22." Same outcome, clearer separation.

### AC Quality

- **S2.1 AC "Plugin still loads in a fresh Claude Code session"** — only verifiable manually. Acceptable for this epic since there's no CI.

- **S2.1 AC `rg -n 'subagent_type.*ruckus' skills/`** — uses `.*` not `.*ruckus:` — would match `subagent_type: "ruckus:code-reviewer"` and also `subagent_type: "anything"` plus `# ruckus` later on the same line. The looser pattern is more conservative.

- **S2.2 AC `rg -n '\.ruckus/' skills/`** — over-matches into the v0.1.4 migration step text. Implementer needs guidance: "AC verified by running the grep, then auditing each match against the allow-list."

- **S2.2 AC "build/SKILL.md ≤ 300 lines after edits"** — current = 294, AC = 300, budget = 6 lines. Realistic. fix/SKILL.md = 299, AC = 300, budget = 1 line. Tight but realistic given character-equivalent substitutions.

- **S2.3 AC concrete path-string check** — the right compromise. Consumer agents have stylistic variations from the canonical preamble; path-string check is the only stable invariant.

- **S2.5 AC "ADR-005 has v0.1.4 footnote added at end of file (parallel to existing v0.1.2 footnote)"** — verified existing v0.1.2 footnote at L52 of ADR-005. Recommend: "v0.1.4 footnote appended below the existing v0.1.2 footnote, separated by a blank line, using the canonical footnote format from epic L383."

- **S2.6 AC `git log --follow .roughly/known-pitfalls.md` shows continuity** — only verifiable post-`git mv`. Excellent forcing function.

- **MISSING AC: no AC verifies `agent-preamble.md` actually parses correctly after the L3–L13 header-comment edit.** If the implementer accidentally breaks the HTML comment delimiters, the body of the comment becomes visible markdown. Recommend S2.3 add an AC: "agent-preamble.md, when rendered, shows only L1 heading and L15–L17 instruction body (the HTML comment is not visible)."

- **MISSING AC: no AC tests the v0.1.2 step still works after S2.2's modifications.** S2.7 step 5 is the v0.1.4 migration test, but the modified v0.1.2 step (now migrating to `.roughly/` directly, skipping `.ruckus/`) needs its own scratch-directory test. Recommend S2.7 add a step 5b.

### Dependencies

- **Phase ordering correct.** S2.1 → S2.2 → (S2.3 ‖ S2.4 ‖ S2.5 ‖ S2.6) → S2.7 is the right shape.
- **Phase 3 disjointness verified.** S2.3 owns `agents/*`, S2.4 owns `skills/setup/templates/*`, S2.5 owns root docs + ADRs, S2.6 owns the dogfood `.ruckus/`.
- **S2.7 allow-list mostly complete** but missing the explicit allow-list entry for `Ruckus` (capital-R prose) in ADR-004 L11/L17, ADR-005 L11/L17/L35, ADR-006, ADR-008 L11. Recommend explicit allow-list entry: "Capital-R `Ruckus` prose in ADR body text (ADR-004 through ADR-008) — historical decision text, preserved per S2.5."
- **S2.5 → S2.6 implicit dependency.** S2.5 updates README/CLAUDE.md to reference `.roughly/` paths. S2.6 actually renames the dogfood `.ruckus/` to `.roughly/`. Phase 3 declares them parallelizable; if S2.5 lands first and S2.6 hasn't run, the README briefly lies. Acceptable but worth noting.

---

## Recommendations (prioritized)

1. **Fix S2.3 line-citation drift** — agent-preamble.md header comment is L3–L13, not L6–L8.
2. **S2.2 spec — partial-failure idempotency.** Add a sub-step about detecting partial migrations (e.g., `.ruckus/.migration-in-progress` marker file) and a clear behavior for re-runs that hit a half-migrated state.
3. **S2.2 spec — abort behavior in conflict prompt.** Document explicitly what `abort` does (recommend: same as v0.1.2 step's abort).
4. **S2.7 allow-list — add `Ruckus` (capital-R) prose in ADR body text** as an explicit allow-list bullet.
5. **README + CHANGELOG migration discoverability** — add a "Migrating from ruckus to roughly" section.
6. **S2.2 boilerplate-rewrite warning** — when point 5b's regex skip-fires, surface a one-line warning in the upgrade summary.
7. **S2.7 add v0.1.2 migration regression test** (step 5b) — modified v0.1.2 step now skips `.ruckus/` and goes straight to `.roughly/`.
8. **S2.5 ADR footnote placement** — explicitly state "appended below existing v0.1.2 footnote, separated by a blank line."
9. **Tighten S2.2 grep ACs** — note exceptions in the AC text or move grep verification into a "verify and disposition allow-listed matches" step.
10. **S2.3 add a render-validation AC** — confirm agent-preamble.md HTML comment delimiters survive the L3–L13 edit.

---

## Files referenced

- `docs/planning/epics/E02-rename-roughly.md`
- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`
- `skills/build/SKILL.md` (294 lines; L96, L195, L209, L250, L255 verified)
- `skills/fix/SKILL.md` (299 lines; L107, L255 verified mixed-content)
- `skills/upgrade/SKILL.md` (125 lines; L17–L19 v0.1.2 step verified, L21/L102/L104 ruckus-version refs verified)
- `skills/setup/SKILL.md` (172 lines; L129/L132 verified, L40 mixed-content verified)
- `skills/setup/templates/CLAUDE.md.template` (L33 verified)
- `skills/setup/templates/known-pitfalls.md.template` (L6 verified)
- `agents/agent-preamble.md` (L17 path verified; **L6–L8 header-comment claim INCORRECT — actual range is L3–L13**)
- `agents/code-reviewer.md` (L18 verified)
- `agents/doc-writer.md` (L22 verified)
- `docs/adrs/ADR-005-versioned-maturity-checks.md` (existing v0.1.2 footnote at L52 verified)
- `docs/adrs/ADR-006-runtime-context-not-baked.md` (existing v0.1.2 footnote at L47 verified)
- `.ruckus/known-pitfalls.md` (L3, L4, L6, L12 content verified)
- `.ruckus/workflow-upgrades` (L1 content verified: `ruckus-version 0.1.3 2026-04-28`)
