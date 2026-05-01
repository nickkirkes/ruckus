# E02 — Rename: ruckus → roughly

**Status:** In Progress (6/7 stories complete — Phase 3 done. S2.1 merged 2026-04-29 PR #18; S2.2 merged 2026-04-29 PR #19 with S2.4 template tokens folded in as a P1 follow-up; line-budget cap fix-up applied 2026-04-30 commit `b2fa658`; S2.3 + S2.6 merged 2026-04-30 PR #20; S2.5 review-passed 2026-04-30 PR #21 (merge pending). Phase 4 remaining: S2.7 release (verification, version bump, finalize CHANGELOG date, tag).)
**Plugin Version:** v0.1.4
**Dependencies:** v0.1.3 (E01 audit follow-up shipped 2026-04-28)
**Validated:** 2026-04-28 (v2 — incorporates `/ruckus:review-epic` Opus findings from same date; second Opus pass on v2 returned `Ready` with three narrow corrections, all applied)

---

## Objective

Rename every reference to "ruckus" across the plugin to "roughly" as a hard-cut release with no aliases or backwards compatibility. The plugin name, slash-command namespace, plugin-installed dotdir, version-line identifier, prose, and templates all migrate. Preserve every ADR-driven behavior unchanged — only identifiers, paths, namespaces, and prose update. Produce a clean v0.1.4 release that is byte-equivalent in behavior to v0.1.3 except for the name.

Decisions confirmed before drafting:
1. Plugin-installed dotdir: `.ruckus/` → `.roughly/`
2. ADRs: temporary edit permission granted; add v0.1.4 footnotes to ADR-004, ADR-005, ADR-006, ADR-008. **ADR body text retains its original `/ruckus:*` and `Ruckus` references as historical decision text** — the footnote provides the rename annotation. ADR-001/002/003/007 contain no "ruckus" mentions; no footnotes needed.
3. Historical PM prompts and archive files preserve their filenames and content
4. README/CONTRIBUTING `ruckus-clone` and `~/.claude/plugins/ruckus/` references update to `roughly`
5. GitHub repo rename and external surface deferred to human after E02 ships

## Canonical Naming (LOCKED)

| Context | Form |
|---|---|
| Code, package, CLI, prose | `roughly` (lowercase) |
| Sentence-start in prose | `Roughly` |
| Brand wordmark | `~roughly` (does NOT appear in this repo) |
| Slash command namespace | `/roughly:*` |

## Discovery Inventory (synthesized from 5 parallel agents, 2026-04-28)

| Territory | Files | Occurrences | Notes |
|---|---|---|---|
| Code & plugin defs (skills/, agents/, .claude-plugin/) | 19 | ~131 | Plugin.json, marketplace.json, all `/ruckus:*` namespace, all `.ruckus/` paths |
| Documentation (README, CLAUDE, CONTRIBUTING, planning/, ADRs) | 6 active + 7 archive + 4 ADRs | ~85 active + ~50 historical | Historical CHANGELOG entries v0.1.0–v0.1.3 preserved verbatim |
| Templates (skills/setup/templates/) | 2 of 4 | 3 | CLAUDE.md.template L33, known-pitfalls.md.template L6 |
| Tests/examples | 0 | 0 | No formal test suite |
| External/hidden + this repo's own dogfood | 2 (`.ruckus/` in repo) | 5 | No `.github/`, no `.husky/`, LICENSE clean |
| Historical implementation plans (`docs/plans/`) | 6 of 8 | ~12 | E01 implementation plans — preserve verbatim |

---

## Stories

### S2.1: Plugin identity and slash-command namespace

**Status:** Complete — merged to main 2026-04-29 via PR #18 (commit `7699d9c`)
**Priority:** P0 (foundation — all other stories cross-reference this)

**Files:**
- `.claude-plugin/plugin.json` — `name` field
- `.claude-plugin/marketplace.json` — `plugins[0].name` field
- `skills/build/SKILL.md` — namespace references at L96, L195, L209, L250, L255
- `skills/fix/SKILL.md` — namespace references at L56, L107, L202, L214, L255, L260
- `skills/review/SKILL.md` — namespace reference at L80
- `skills/review-plan/SKILL.md` — frontmatter description L3 (two `/ruckus:*` instances), L19
- `skills/setup/SKILL.md` — L40, L149, L169, L170
- `skills/upgrade/SKILL.md` — L18, L25, L39 (`subagent_type: "ruckus:code-reviewer"` example)

**Context:**

The plugin's identity is set in two places: `.claude-plugin/plugin.json` (`"name": "ruckus"`) and `.claude-plugin/marketplace.json` (`"plugins": [{"name": "ruckus", ...}]`). Both are read by Claude Code at plugin load time and define the slash-command namespace. Every `/ruckus:*` reference in skill bodies — used for subagent dispatch and user-facing instructions — must match the plugin name exactly, or dispatch fails silently.

Per ADR-001, `review-plan` is dispatched as a blocking subagent. The dispatch pattern uses the slash-command namespace (e.g., `/ruckus:review-plan` in build/SKILL.md L96). Renaming the namespace preserves the ADR-001 dispatch mechanism — only the identifier changes.

Per ADR-004, UI work is conditional on a per-task flag, not a separate command. This story does not reintroduce `/roughly:build-ui` — that pattern remains forbidden (verified at Exit Criteria).

Per ADR-008, only epic-reviewer uses Opus; this rename does not touch model selection.

The `subagent_type: "ruckus:code-reviewer"` example in `skills/upgrade/SKILL.md` L39 is illustrative documentation, not a functional dispatch — but it must match plugin name post-rename for the documentation to be accurate.

**Current behavior (excerpts):**

```json
// .claude-plugin/plugin.json
{ "name": "ruckus", "description": "...", "version": "0.1.3" }
```

```json
// .claude-plugin/marketplace.json — plugins[0].name
{ "name": "ruckus", "source": "./", "description": "..." }
```

```markdown
// skills/build/SKILL.md L96
Dispatch `/ruckus:review-plan` as a blocking subagent call.

// skills/build/SKILL.md L250 (mixed-content line — namespace half is S2.1, prose `Ruckus` half is S2.2)
> "CLAUDE.md is missing [fields]. This reduces the quality of every Ruckus skill. Run `/ruckus:setup` to fix..."
```

**Target behavior:** Replace `"ruckus"` with `"roughly"` in plugin.json and marketplace.json `name` fields. Replace every `/ruckus:*` slash-command reference with `/roughly:*` in the listed skills. Replace `subagent_type: "ruckus:code-reviewer"` with `subagent_type: "roughly:code-reviewer"` at upgrade/SKILL.md L39.

**Mixed-content line coordination:** Several lines (build L250, fix L255, setup L40) contain both a `/ruckus:*` token (S2.1's responsibility) and a prose `Ruckus` token (S2.2's responsibility). When implemented sequentially, S2.1 edits the namespace token first; S2.2 then edits the prose token. If parallelized in worktrees, the second-merging branch will have a textual conflict on these lines and must hand-resolve. Recommend serializing S2.1 → S2.2 to avoid conflicts.

**Implementer instruction for S2.1:** On the three mixed-content lines, edit ONLY the `/ruckus:*` namespace token. DO NOT do a wholesale line replacement that also rewrites the prose `Ruckus` — leave that for S2.2. If you accidentally rewrite the whole line, revert and edit only the namespace token. (S2.2 has a corresponding "no-op if S2.1 already replaced both" clause.)

**ADR constraints:**
- ADR-001: Preserve the blocking-subagent dispatch mechanism for review-plan; only the namespace identifier changes
- ADR-004: Do not introduce `/roughly:build-ui`
- ADR-008: Do not change model selection on any agent

**Cross-reference impacts:**
- This story changes the namespace; S2.5 (documentation) updates README/CONTRIBUTING/CLAUDE.md which contain `/ruckus:*` examples. S2.5 must run after S2.1.
- Templates (S2.4) contain `/ruckus:build` and `/ruckus:fix` references — covered by S2.4.
- Agent files do NOT contain `/ruckus:*` namespace references (verified) — only `.ruckus/` paths, which are S2.3.
- ADR-004 and ADR-005 have inline `/ruckus:*` references in their decision text; these are PRESERVED as historical (S2.5 adds footnotes, does not rewrite ADR body text).

**Acceptance Criteria:**
- [ ] `.claude-plugin/plugin.json` has `"name": "roughly"`
- [ ] `.claude-plugin/marketplace.json` has `"name": "roughly"` in `plugins[0]`
- [ ] Zero `/ruckus:` namespace references in `skills/**/SKILL.md` (verified via `rg -n '/ruckus:' skills/`); exception allowed inside the v0.1.4 upgrade-migration step (S2.2's territory) — that step intentionally references the legacy namespace for migration prompts
- [ ] Zero `subagent_type:.*ruckus:` references in skills (verified via `rg -n 'subagent_type.*ruckus' skills/`)
- [ ] Plugin still loads in a fresh Claude Code session: running `claude --plugin-dir <repo>` from a fresh project, `/roughly:setup` appears in autocomplete and dispatches without error

---

### S2.2: Skills `.ruckus/` migration, version-line rename, skill prose, and v0.1.4 upgrade-migration step

**Status:** Complete — PR #19 review passed 2026-04-29 (head `4ed9b2e`)
**Priority:** P0 (foundation — depends on S2.1; gates S2.3 and S2.4)

**Files:**

Path references (`.ruckus/` → `.roughly/`):
- `skills/build/SKILL.md` — L144, L231, L237, L254, L256, L259
- `skills/fix/SKILL.md` — L40, L153, L236, L242, L259, L261, L264
- `skills/setup/SKILL.md` — L31, L42, L86, L102, L105, L127, L143, L150, L163, L164
- `skills/upgrade/SKILL.md` — L17, L18, L19 (multi-instance migration logic), L21, L24, L33, L37, L41, L102
- `skills/review/SKILL.md` — L28, L38, L71
- `skills/review-plan/SKILL.md` — L17, L19, L89
- `skills/review-epic/SKILL.md` — L34
- `skills/build/implementer-prompt.md` — L21

Version-line identifier (`ruckus-version` → `roughly-version`) — **5 references, not 4**:
- `skills/setup/SKILL.md` — L129 (file content example), L132 (re-run/enrich logic)
- `skills/upgrade/SKILL.md` — L21 (read), L102 (write instruction), L104 (write content)

Skill-body prose `Ruckus` → `Roughly` and `ruckus` → `roughly`:
- `skills/setup/SKILL.md` — L6 (`# Ruckus Setup`), L8 (intro paragraph), L40 (legacy-installation prompt — also has S2.1 namespace work; S2.2 owns the prose token), L43, L102, L143, L158, L171
- `skills/upgrade/SKILL.md` — L6 (heading), L8 (intro paragraph), L17 (migration prompt prose)
- `skills/review-epic/SKILL.md` — L60 ("Ruckus epic-reviewer" → "Roughly epic-reviewer")
- `skills/build/SKILL.md` — L250 ("every Ruckus skill" prose — S2.1 owns the `/ruckus:setup` token on the same line)
- `skills/fix/SKILL.md` — L255 ("every Ruckus skill" prose — S2.1 owns the `/ruckus:setup` token on the same line)

New v0.1.4 upgrade-migration step (added to upgrade/SKILL.md after the v0.1.2 step at L17–L19):
- `skills/upgrade/SKILL.md` — STEP 1, new sub-step

**Context:**

The plugin-installed dotdir `.ruckus/` is created in user projects by setup and contains `.ruckus/known-pitfalls.md` and `.ruckus/workflow-upgrades`. Per ADR-006, skills read project context from these paths at runtime — not baked into skill text. The rename moves the dotdir to `.roughly/`, mirroring v0.1.2's `docs/claude/` → `.ruckus/` precedent.

The `.ruckus/workflow-upgrades` file uses a `ruckus-version <ver> <date>` line format. The line format identifier becomes `roughly-version`. **All five references — setup L129 and L132, upgrade L21, L102, L104 — must move together**. If they fall out of sync, version detection fails and upgrade routing breaks.

Per ADR-005, maturity-check IDs (e.g., `investigator-v1-added`) are versioned. None of the existing check IDs (`investigator-v1`, `pitfalls-organized-v1`) embed "ruckus", so no version bump is required by the rename. The version-line identifier `ruckus-version` is plugin metadata, not a check ID — renaming it to `roughly-version` does not bump any check.

**v0.1.2 migration step modification (upgrade/SKILL.md L17–L19):** The existing step migrates `docs/claude/` → `.ruckus/`. Modify to migrate to `.roughly/` directly (no `.ruckus/` intermediate). Concrete substring replacements within the existing step's text:
- L17 prompt text: `Ruckus v0.1.2 renamed `docs/claude/` to `.ruckus/`` → `Roughly renamed `docs/claude/` to `.roughly/``
- L17/L18 abort message: every `.ruckus/` → `.roughly/`; the `/ruckus:upgrade` token is S2.1's territory
- L19 logic block: every `.ruckus/known-pitfalls.md` substring → `.roughly/known-pitfalls.md`; every `.ruckus/workflow-upgrades` → `.roughly/workflow-upgrades`; the post-move CLAUDE.md substitution writes `.roughly/`. Preserve the existing CLAUDE.md merge logic verbatim except for these path strings.

**New v0.1.4 migration step (added immediately after the modified v0.1.2 step at L19):**

The v0.1.4 step detects an existing `.ruckus/` directory in the user's project and migrates it to `.roughly/`. Detailed behavior (10 points):

1. **Detection and git/non-git method choice:** If `.ruckus/` directory does not exist, skip this step entirely. If it exists, proceed. Detect git status with `git rev-parse --git-dir 2>/dev/null` (silent failure means non-git). Use `git mv` inside a git repo; use plain `mv` otherwise. Do NOT try `git mv` and fall back — the failure output is confusing to users.

2. **Marker file (start-of-migration):** Before any file moves, write a marker at `.ruckus/.migration-in-progress` containing the current ISO date and the plugin version performing the migration (one line, plain text). The marker is the canonical signal that a migration started but may not have finished. The marker is removed at step 9 once the migration is complete.

    **If the marker write fails** (read-only filesystem, restrictive permissions, disk full): abort the v0.1.4 migration step immediately with the message `"Cannot write marker to .ruckus/.migration-in-progress — check directory permissions or filesystem state."` No file moves have been attempted at this point, so no partial state is created. The user can fix permissions and re-run `/roughly:upgrade`.

3. **Conflict check (or partial-failure resume):** If `.roughly/` already exists alongside `.ruckus/`:
    - **If `.ruckus/.migration-in-progress` exists**, classify as a half-completed migration. Skip the conflict prompt; resume from step 5 (move). Steps 5–7 must be implemented as idempotent (skip-if-already-moved, skip-if-already-rewritten) so resume is safe.
    - **Otherwise**, prompt: `"Both .ruckus/ and .roughly/ exist (no in-progress marker). Show diff and ask which to keep? (diff / keep .roughly / keep .ruckus / abort)"`. On `keep .roughly`: leave `.roughly/` as-is, move only files from `.ruckus/` that don't exist in `.roughly/`, then continue at step 5. On `keep .ruckus`: replace `.roughly/` with `.ruckus/` content (move-and-overwrite), then continue. On `diff`: show file-by-file diff and let user choose per file. **On `abort`:** stop the upgrade entirely with the same message pattern as the v0.1.2 step's abort: `"Migration is required before upgrading — re-run /roughly:upgrade when ready to migrate."` Do NOT remove the marker on abort. (For the conflict-prompt path: marker was never written, so re-running re-prompts on every conflict-no-marker run until the user picks a non-abort option. For the partial-failure path with a marker present: re-running resumes from step 5 without re-prompting.)

4. **`docs/claude/` AND `.ruckus/` co-existence:** If both legacy dirs exist, the v0.1.2 step (above) ran first and wrote to `.roughly/`. The v0.1.4 step then encounters non-empty `.roughly/` per the conflict check at step 3. This case is the same as step 3.

5. **Move (idempotent):** Move `.ruckus/known-pitfalls.md` → `.roughly/known-pitfalls.md` and `.ruckus/workflow-upgrades` → `.roughly/workflow-upgrades` using the command from step 1. If the source does not exist (already moved), skip that move silently. If the destination exists and the source also exists (resume case after a partial move), keep the destination, delete the source.

6. **Content rewrites inside the moved files (idempotent):**
    - In `.roughly/workflow-upgrades`: rename the version-line identifier in-place from `ruckus-version` to `roughly-version`. Preserve the existing version number and date — STEP 6 of upgrade later updates the version digits. Do NOT double-process: this step renames the identifier only. If the line already reads `roughly-version` (resume case), skip silently.
    - In `.roughly/known-pitfalls.md`: rewrite the boilerplate header line. **Match anchored to start-of-line:** `^Pitfalls.*\/ruckus:build.*\/ruckus:fix.*$` — replace `/ruckus:build` and `/ruckus:fix` with `/roughly:build` and `/roughly:fix`. Leave all user-authored pitfall entries unchanged. **On non-match (line was customized or commands reordered):** skip rewrite silently AND log a one-line warning to the upgrade summary: `"Custom boilerplate line in .roughly/known-pitfalls.md preserved verbatim — check for stale /ruckus:* tokens manually."` On resume case where the line already reads `/roughly:*`, the regex won't match (no `\/ruckus:` substring); the silent-skip + warning behavior is acceptable; the warning will be a false alarm but harmless.

7. **Update root CLAUDE.md path references (literal substring, with counts):** Use literal-substring match (NOT regex) to replace `.ruckus/known-pitfalls.md` → `.roughly/known-pitfalls.md` and `.ruckus/workflow-upgrades` → `.roughly/workflow-upgrades` in the root `CLAUDE.md`. Display before/after match counts in the upgrade summary, e.g., `"Updated 2 path references in CLAUDE.md (1 known-pitfalls.md, 1 workflow-upgrades)."` If no matches (user customized — already on `.roughly/` paths or missing entirely), display `"No .ruckus/ path references in root CLAUDE.md — skipped."` and continue.

8. **User-extra files in `.ruckus/` (interactive prompt):** After the standard moves at step 5, list any files remaining in `.ruckus/` beyond the marker file (which step 2 created). If the list is non-empty, prompt: `"Files [X, Y, Z] exist in .ruckus/ beyond the standard known-pitfalls.md and workflow-upgrades. Move them to .roughly/ as well? (move / leave for me to handle)"`. On `move`: move all listed files to `.roughly/` (using the same git-vs-plain-mv command from step 1). On `leave`: print the path and continue without moving.

9. **Cleanup:** Remove the marker file `.ruckus/.migration-in-progress` **regardless of whether `.ruckus/` is also removed**. The marker indicates "migration in progress"; once the standard moves are done and the user has dispositioned any extras at step 8, the migration is complete and the marker must not persist (a stale marker would trigger a partial-failure-resume at step 3 on the next run, even though no work remains). If `.ruckus/` is empty (or contains only the marker, which step 9 has just removed), remove the directory. If `.ruckus/` is still non-empty after the marker delete (user chose `leave` at step 8), do NOT remove `.ruckus/`; print: `"Files remain in .ruckus/. Move or delete them when ready."` and continue.

10. **Idempotency:** With the marker mechanism, a successful migration removes the marker at step 9; re-running the upgrade finds no `.ruckus/` and skips this step at step 1. A mid-migration failure leaves the marker in place; re-running detects the marker at step 3 and resumes from step 5.

**Current behavior (excerpts):**

```markdown
// skills/setup/SKILL.md L86
Create `.ruckus/` directory if it doesn't exist.

// skills/setup/SKILL.md L129
ruckus-version [version from plugin.json] [today's date YYYY-MM-DD]

// skills/setup/SKILL.md L132
If the file already exists (re-run / enrich mode): update or insert the `ruckus-version` line at the top, preserving all other entries.

// skills/upgrade/SKILL.md L102
Update the `ruckus-version` line in `.ruckus/workflow-upgrades` to match the current plugin version.
```

**Target behavior:** Path renames (`.ruckus/` → `.roughly/`), version-line identifier renames (`ruckus-version` → `roughly-version`), prose `Ruckus` → `Roughly` per the file/line enumeration above. Modify the v0.1.2 migration step per the substring rules above. Add the new v0.1.4 migration step per the 10-point spec above.

**Coordination with S2.1:** Both stories modify `skills/build/SKILL.md`, `skills/fix/SKILL.md`, `skills/setup/SKILL.md`, and `skills/upgrade/SKILL.md`, but on disjoint lines (S2.1 owns namespace tokens; S2.2 owns paths, version-line identifiers, and skill prose). On lines containing both kinds of token (build L250, fix L255, setup L40), S2.1 edits the namespace half; S2.2 edits the prose/path half. If parallelized in worktrees, hand-resolve textual conflicts on these lines. **Recommend serial execution S2.1 → S2.2.**

**ADR constraints:**
- ADR-005: No maturity check IDs are renamed; the version-line identifier is metadata only. No version bumps.
- ADR-006: Runtime context loading preserved — skills still read CLAUDE.md and `.roughly/known-pitfalls.md` at runtime.
- ADR-001: review-plan dispatch mechanism preserved (S2.1's namespace change does not alter dispatch verb).

**Cross-reference impacts:**
- S2.3 (agents) updates the same `.ruckus/known-pitfalls.md` path in agent files — must use the same target path
- S2.4 (templates) updates `skills/setup/templates/CLAUDE.md.template` L33 and `known-pitfalls.md.template` L6 — must use the same target path and namespace
- S2.6 (this repo's own dogfood) renames `.ruckus/` → `.roughly/` in the plugin source repo
- S2.5 (docs) covers root README/CLAUDE.md/CONTRIBUTING.md prose only; skill-internal prose is owned here

**Line budget and editing constraint:** Current build/SKILL.md = 294 lines, fix/SKILL.md = 299 lines (1-line headroom), upgrade/SKILL.md = 125 lines.

**Constraint for build/SKILL.md and fix/SKILL.md: in-place character substitutions ONLY — no line additions, no line removals.** The dotdir rename adds ~1 char per occurrence and version-line/prose renames are character-equivalent or shrink; markdown line-length is not enforced, so net line count stays flat. Any structural change (added bullet, new sentence, reflowed paragraph) requires explicit PM approval — the fix/SKILL.md budget allows zero new lines.

upgrade/SKILL.md is exempt from the in-place constraint because it gains the new 10-point v0.1.4 migration step (~25–35 lines after this expansion). Final upgrade/SKILL.md is expected at ~150–160 lines, well under the 300-line cap.

**Acceptance Criteria:**

Grep-based ACs are **verified by running the grep then auditing each match against the allow-list** — they are not pure zero-match assertions. The allow-list for `.ruckus/` and `\bRuckus\b` and `/ruckus:` matches inside `skills/`:
- Lines INSIDE the v0.1.2 migration prompt text in `skills/upgrade/SKILL.md` (intentionally name `.ruckus/` and `Ruckus` for the migration prose).
- Lines INSIDE the new v0.1.4 migration step in `skills/upgrade/SKILL.md` (intentionally name `.ruckus/` to detect legacy installations and `/ruckus:build`/`/ruckus:fix` in the regex match-pattern at point 6).

Outside those allow-listed regions:

- [x] Zero `.ruckus/` references in `skills/**/SKILL.md` and `skills/build/implementer-prompt.md` outside allow-list (verified 2026-04-30: all matches are inside the v0.1.4 migration step at upgrade/SKILL.md L21–58 OR in the NEW pre-flight migration checks added to build/fix/review/review-plan/review-epic/setup — see Post-merge audit below for scope-expansion note)
- [x] Zero `ruckus-version` references in skills outside the v0.1.4 migration step's content-rewrite spec (verified 2026-04-30: only match is at upgrade/SKILL.md L47 inside the migration step; setup writes `roughly-version`)
- [x] Zero capital-R `Ruckus` prose in skills (verified 2026-04-30: `rg -n '\bRuckus\b' skills/` returns zero matches)
- [x] `setup/SKILL.md` writes to `.roughly/` directory and uses `roughly-version` line format (verified 2026-04-30 at setup/SKILL.md L132 — line shifted from L129 due to S2.2 edits)
- [x] `upgrade/SKILL.md` v0.1.2 migration step now migrates `docs/claude/` → `.roughly/` directly (verified 2026-04-30 by inspecting upgrade/SKILL.md L8–18 v0.1.2 step)
- [x] `upgrade/SKILL.md` has a NEW v0.1.4 migration step implementing the 10-point spec (verified 2026-04-30 at upgrade/SKILL.md L21–58)
- [ ] **build/SKILL.md = 294 lines after edits — FAILED.** Actual: **296 lines** (+2). Cause: a NEW pre-flight migration check was added at L19 (scope expansion not in S2.2 spec). Within CLAUDE.md's 300-line cap (296 ≤ 300), but violates the explicit S2.2 in-place-only constraint. See Post-merge audit below.
- [x] **fix/SKILL.md ≤ 300 lines (CLAUDE.md cap)** — FIX-UP APPLIED 2026-04-30. Original: 301 lines (cap violation). Post-fix: **299 lines** (under cap with 1-line headroom). Trim was a 2-line merge in the MATURITY CHECKS section (Format-per-entry sentence absorbed into the preceding "Check IDs are versioned" paragraph) — purely prose collapse, no behavior change. The original AC of "= 299 lines after edits (in-place character substitutions only)" is now satisfied as 299 lines, but technically violated mid-flight by the S2.2 implementation; the post-merge fix-up restores compliance.
- [x] upgrade/SKILL.md ≤ 165 lines after edits (verified 2026-04-30: actual is 164 lines)
- [x] No maturity-check IDs were renamed (ADR-005 compliance — verified 2026-04-30: `investigator-v1`, `pitfalls-organized-v1` unchanged)
- [x] review-plan still dispatched as blocking subagent (ADR-001 compliance — verified 2026-04-30 at build/SKILL.md L98 and fix/SKILL.md L109; lines shifted from L96/L107 in original AC due to other edits, behavior preserved verbatim)
- [x] `subagent_type: "roughly:code-reviewer"` confirmed at upgrade/SKILL.md L78 (line shifted from L39 due to v0.1.4 migration step insertion)
- [x] **Mixed-content lines:** verified 2026-04-30 — no `\bRuckus\b` prose remains in skills; all namespace tokens are `/roughly:*`. The originally-cited line numbers (build L250, fix L255, setup L40) shifted, but the underlying behavior verifies clean.

---

**Post-merge audit (2026-04-30):**

S2.2 PR #19 merged successfully and is functionally complete: the 10-point v0.1.4 migration step is implemented, the v0.1.2 step skips `.ruckus/` intermediate, and all residue checks pass against the allow-list. Two findings worth flagging:

1. **Beneficial scope expansion — pre-flight migration checks added to 5 additional skills.** The implementer added a 1-line pre-flight migration check at the top of `skills/build/SKILL.md` L19, `skills/fix/SKILL.md` L19, `skills/review/SKILL.md` L18, `skills/review-plan/SKILL.md` L13, `skills/review-epic/SKILL.md` L17, and `skills/setup/SKILL.md` L39 — pattern: detect legacy `.ruckus/.migration-in-progress`, `.ruckus/known-pitfalls.md`, or `.ruckus/workflow-upgrades` and abort with a redirect to `/roughly:upgrade`. This protects v0.1.3 users who install the new plugin without first running `/roughly:upgrade` — they get a clear error instead of a confusing failure mode. **Recommendation: accept; it is genuinely user-protective and aligned with the hard-cut rename UX. Document in CHANGELOG (handled by S2.7).**

2. **Line-budget cap violation — fix/SKILL.md was 301 lines (CLAUDE.md cap is 300).** The pre-flight check pushed fix/SKILL.md from 299 → 301. **RESOLVED 2026-04-30** by a small in-place trim in the MATURITY CHECKS section: collapsed the standalone "Format per entry: …" line into the preceding "Check IDs are versioned" paragraph as a trailing sentence. Net change: -2 lines (301 → 299), no behavior change, residue audit unchanged, all 10 stage/maturity/abort headers intact. build/SKILL.md remains at 296 (within the 300 cap; no fix-up needed) — the in-place-only S2.2 AC is technically violated by the +2 from the pre-flight check, but the cap is satisfied.

---

### S2.3: Agent preamble sync (`.ruckus/` → `.roughly/`) ✅

**Status:** Complete — PR #20 review passed 2026-04-30 (commit `23c1846` for the sync, `f1bb2f1` for the post-implementation pitfalls capture). Merge pending.
**Priority:** P1 (depends on S2.2 dotdir decision; can run in parallel with S2.2 since file sets are disjoint)

**Files:**
- `agents/agent-preamble.md` — L17 (canonical path reference) AND the HTML comment block at L3–L13 (see Context for the targeted sub-edit; the `<!--` at L3 and `-->` at L13 delimiters MUST be preserved verbatim)
- `agents/code-reviewer.md` — L18
- `agents/discovery.md` — L22
- `agents/epic-reviewer.md` — L18
- `agents/investigator.md` — L22
- `agents/silent-failure-hunter.md` — L18
- `agents/doc-writer.md` — L22 (write-target reference)

NOT modified:
- `agents/static-analysis.md` — does not reference the path; documented exception

**Context:**

Per CLAUDE.md and ADR-003 design pattern, `agents/agent-preamble.md` is the canonical sync reference for the project-context-loading instruction. Each consumer agent inlines the preamble text in its Process step 1 (no auto-include — Claude Code plugins have no template engine). When the canonical preamble is updated, every consumer must be manually synced.

The current preamble path line reads:
```
2. **.ruckus/known-pitfalls.md** -- known issues to avoid repeating
```

Each preamble-inlining consumer (code-reviewer, discovery, epic-reviewer, investigator, silent-failure-hunter — 5 agents) has a parallel path reference. doc-writer references the same path string but as a write-target description, not as preamble text.

**Header-comment update (within the L3–L13 HTML comment):** Verified actual structure of `agents/agent-preamble.md`:
- L3 `<!--` (open delimiter)
- L5–L8 manual-sync target list (5 preamble-inlining consumers: code-reviewer, silent-failure-hunter, investigator, discovery, epic-reviewer, plus the inlined templates in build/SKILL.md and fix/SKILL.md Stage 5b)
- L9–L13 exceptions paragraph (static-analysis and doc-writer described in prose)
- L13 `-->` (close delimiter)

**Do not add doc-writer to the L5–L8 manual-sync list** — that list is for agents using the preamble inlining pattern, which doc-writer does not. Instead, add a new sentence to the doc-writer description in the L9–L13 exceptions paragraph clarifying that the path string at doc-writer L22 still requires sync when the path changes (despite the different pattern). Suggested wording for the appended sentence: `"Path-string sync still required for doc-writer L22 (write-target reference) whenever the known-pitfalls.md path changes."` This keeps the preamble-pattern semantic separation clean (5 inliners; doc-writer's pattern truly is different) while ensuring future contributors don't miss doc-writer's L22 when the path changes again.

**Critical: do not break the HTML comment delimiters.** All edits within L3–L13 stay between `<!--` and `-->`. After the edit, the rendered markdown should show only L1 heading + L15–L17 instruction body — the HTML comment must remain invisible to the renderer.

The two documented preamble-pattern exceptions are static-analysis (uses CLAUDE.md commands only, not pitfalls — has zero "ruckus" references) and doc-writer (writes to known-pitfalls.md as a target). doc-writer's L22 path-string update is in scope here; the preamble drift check in upgrade/SKILL.md (modified by S2.2) excludes both exceptions.

**Current behavior:**

```markdown
// agents/agent-preamble.md L17
2. **.ruckus/known-pitfalls.md** -- known issues to avoid repeating

// agents/code-reviewer.md L18 (and parallel lines in 4 other consumers)
1. **Read project context** — CLAUDE.md and .ruckus/known-pitfalls.md

// agents/doc-writer.md L22
- `.ruckus/known-pitfalls.md` — Add new pitfalls discovered during development
```

**Target behavior:** Replace `.ruckus/known-pitfalls.md` with `.roughly/known-pitfalls.md` in agent-preamble.md L17 and the parallel lines in the 6 consumer agents. Within the L3–L13 HTML comment in agent-preamble.md, leave the L5–L8 manual-sync target list unchanged (5 inliners + 2 inlined templates) and append the doc-writer path-sync sentence to the existing doc-writer description in the L9–L13 exceptions paragraph. Preserve the `<!--` and `-->` delimiters.

**ADR constraints:**
- ADR-003: The shared-reference pattern requires manual sync. This story is exactly that sync operation.
- ADR-006: Runtime context-loading semantics preserved — only the path string changes.

**Cross-reference impacts:**
- The upgrade skill's preamble drift check (S2.2 territory) is the verification mechanism. After this story merges, run `/roughly:upgrade` (post-S2.1) on a test project; the drift check should report no drift.
- The implementer-prompt.md L21 line is S2.2 territory (build-pipeline file, not an agent), but must use the same target path `.roughly/known-pitfalls.md` for consistency.

**Acceptance Criteria:**

- [x] `agents/agent-preamble.md` references `.roughly/known-pitfalls.md` (verified 2026-04-30 at L18 — line shifted from cited L17 because the doc-writer path-sync sentence extended the HTML comment block by one line; path-string check is the stable invariant)
- [x] Each of the 6 consumer agents (code-reviewer, discovery, epic-reviewer, investigator, silent-failure-hunter, doc-writer) references `.roughly/known-pitfalls.md` at the cited line numbers (verified 2026-04-30: code-reviewer L18, discovery L22, epic-reviewer L18, investigator L22, silent-failure-hunter L18, doc-writer L22)
- [x] `agents/agent-preamble.md` exceptions paragraph contains an appended sentence about doc-writer path-string sync (verified 2026-04-30 at L14 — sentence sits inside the HTML comment, immediately before the `-->` close delimiter; the L5–L8 manual-sync target list is unchanged)
- [x] **HTML comment delimiters preserved.** Verified 2026-04-30: `<!--` at L3, `-->` at end of L14 (post-extension). The HTML comment block now spans L3–L14 (was L3–L13 pre-edit). Rendered output is L1 heading + L16–L18 instruction body only.
- [x] static-analysis.md remains untouched (verified 2026-04-30: zero `ruckus`/`roughly` matches in static-analysis.md)
- [x] Zero `.ruckus/known-pitfalls.md` references in `agents/` (verified 2026-04-30: `rg -n '\.ruckus/known-pitfalls' agents/` returns zero)
- [x] Zero capital-R `Ruckus` prose in `agents/` (verified 2026-04-30: `rg -n '\bRuckus\b' agents/` returns zero)
- [x] The path string `.roughly/known-pitfalls.md` appears at the cited line of each consumer (verified 2026-04-30 — see consumer table above)

**Post-merge audit (2026-04-30):**

S2.3 PR #20 also includes scope-expansion commits beyond the original spec:

1. **`f1bb2f1` — pitfalls capture (beneficial).** The implementation agent recorded two pitfalls discovered during S2.3 implementation in `.roughly/known-pitfalls.md`. This is the documented wrap-up flow per ADR-006 / build-fix Stage 8 — appropriate use of the maturity-driven knowledge capture loop.
2. **`a717c47` + `1a01f55` — Stop-hook structural verify-all (beneficial scope expansion).** A new `.claude/hooks/verify-all.sh` script was added (58 lines) along with `.claude/settings.json` wiring. The hook fires after every Claude turn and reports drift on: stale `.ruckus/known-pitfalls` references in `agents/`, skill bodies > 300 lines, agent bodies > 500 words, and HTML comment integrity in agent-preamble.md. Per ADR-005 maturity-check `stop-hook-v1`, this is exactly the kind of optional self-verification the plugin is supposed to opt into when verification is robust enough — this commit represents that opt-in for the dogfood repo. Out of S2.3's spec scope but on-pattern. **Recommendation: accept; document in CHANGELOG as a contributor-facing addition.**

These scope expansions do not invalidate any S2.3 AC and do not change any ADR-driven behavior. The Stop hook is dogfood-only (it checks for the plugin repo's own structural conventions and is a no-op outside the repo per its `cd "$ROOT"` guard).

---

### S2.4: Templates rename (`.ruckus/` → `.roughly/`, `/ruckus:*` → `/roughly:*`) ✅

**Status:** Complete — folded into S2.2 PR #19 as a P1 follow-up (commit `536100f`, 4th commit on the merge). The S2.2 author pulled both template token edits forward because the S2.2 write-target change (rendered output going to `.roughly/known-pitfalls.md`) made the unmigrated templates an immediate regression for new `/roughly:setup` runs — the deferral to S2.4 per blast radius would have shipped a broken first-install experience. Verified 2026-04-30 against `main`: all 5 ACs satisfied. No separate PR needed for this story; closing with a docs-only status update.
**Priority:** P1 (depends on S2.1 namespace decision and S2.2 dotdir decision; parallelizable with S2.2/S2.3 since files disjoint)

**Files:**
- `skills/setup/templates/CLAUDE.md.template` — L33 (`.ruckus/known-pitfalls.md` path reference)
- `skills/setup/templates/known-pitfalls.md.template` — L6 (`/ruckus:build` and `/ruckus:fix` command references)

NOT modified:
- `skills/setup/templates/claudeignore.template` — verified zero "ruckus" references
- `skills/setup/templates/settings.json.template` — verified zero "ruckus" references

**Context:**

Templates ship into user projects when setup runs. They are populated by `skills/setup/SKILL.md` STEP 2 with placeholder substitution. Currently:
- `CLAUDE.md.template` L33 reads `- Known pitfalls: .ruckus/known-pitfalls.md` — lands in every user's root CLAUDE.md
- `known-pitfalls.md.template` L6 reads ``Pitfalls discovered through development. Updated by `/ruckus:build` and `/ruckus:fix` wrap-up stages.`` — lands in every user's `.ruckus/known-pitfalls.md` (which becomes `.roughly/known-pitfalls.md` after S2.2)

Post-rename, new projects running `/roughly:setup` should receive `.roughly/known-pitfalls.md` populated with `/roughly:build` and `/roughly:fix` references. The S2.2 v0.1.4 upgrade-migration step handles existing user projects (rewrites the boilerplate L6 line in migrated content per S2.2's spec); this story handles new-project content via the templates.

**Current behavior:**

```markdown
// skills/setup/templates/CLAUDE.md.template L33
- Known pitfalls: .ruckus/known-pitfalls.md

// skills/setup/templates/known-pitfalls.md.template L6
Pitfalls discovered through development. Updated by `/ruckus:build` and `/ruckus:fix` wrap-up stages.
```

**Target behavior:**

```markdown
// CLAUDE.md.template L33
- Known pitfalls: .roughly/known-pitfalls.md

// known-pitfalls.md.template L6
Pitfalls discovered through development. Updated by `/roughly:build` and `/roughly:fix` wrap-up stages.
```

**ADR constraints:**
- ADR-006: Templates use `{{PLACEHOLDER}}` for project-specific values. The plugin name `roughly` is NOT a placeholder — it's a literal value (treat like `git` or `npm`). No placeholderization needed. Existing placeholders (`{{PROJECT_NAME}}`, `{{DOMAIN_DESCRIPTION}}`, etc.) are untouched.

**Cross-reference impacts:**
- S2.2's setup/SKILL.md L105 reads CLAUDE.md.template and writes to `.roughly/known-pitfalls.md`. Both stories must agree on the target dotdir name.
- S2.2's v0.1.4 upgrade-migration step rewrites the L6 boilerplate line inside existing user known-pitfalls.md files (per S2.2 spec point 5b). This story ensures fresh setups also produce content with the new naming.

**Acceptance Criteria:**

- [x] `skills/setup/templates/CLAUDE.md.template` L33 reads `- Known pitfalls: .roughly/known-pitfalls.md` (verified 2026-04-30 against `main`)
- [x] `skills/setup/templates/known-pitfalls.md.template` L6 references `/roughly:build` and `/roughly:fix` (verified 2026-04-30 against `main`)
- [x] Zero "ruckus" references in `skills/setup/templates/` (verified 2026-04-30: `rg -i ruckus skills/setup/templates/` returns zero)
- [x] `claudeignore.template` and `settings.json.template` are unchanged (verified 2026-04-30: mtime predates commit `536100f`; no edits in S2.2 PR diff)
- [x] Existing `{{PLACEHOLDER}}` markers in templates are unchanged (verified 2026-04-30: `{{PROJECT_NAME}}`, `{{DOMAIN_DESCRIPTION}}`, `{{STACK_SUMMARY}}`, `{{BUILD_COMMAND}}`, `{{TYPE_CHECK_COMMAND}}`, `{{TEST_COMMAND}}`, `{{FORMATTER_COMMAND}}`, `{{CONVENTIONS}}`, `{{ARCHITECTURE_NOTES}}`, `{{CROSS_BOUNDARY_NOTES}}`, `{{ADR_LOCATION}}` all present in CLAUDE.md.template; `{{FORMATTER_COMMAND}}` in settings.json.template; `{{PROJECT_NAME}}`, `{{DOMAIN_DESCRIPTION}}` in known-pitfalls.md.template)

---

### S2.5: Documentation prose, ADR footnotes ✅

**Status:** Complete — PR #21 review passed 2026-04-30 (commit `0ae8511` for the rename + ADR footnotes; `4ff8e5a` for pitfalls capture; `7b6ad8f` for repo-relative path fix in plan verify commands). Merge pending.
**Priority:** P2 (parallelizable with S2.2/S2.3/S2.4 after S2.1; not validated)

**Files:**

Active docs (full prose + CLI rename):
- `README.md` — title, opening paragraph, install commands, clone path (`~/.claude/plugins/ruckus/` → `~/.claude/plugins/roughly/`), all `/ruckus:*` references in code blocks, all prose `Ruckus`/`ruckus` references
- `CLAUDE.md` — title, opening paragraph, structure table prose, `/ruckus:setup` reference at L14, clone path at L29
- `CONTRIBUTING.md` — title, prose, clone path references (`/path/to/your/ruckus-clone` → `roughly-clone`), `/ruckus:*` examples

ADR footnotes (annotation only — body text preserved):
- `docs/adrs/ADR-004-ui-conditional-not-forked.md` — add v0.1.4 footnote
- `docs/adrs/ADR-005-versioned-maturity-checks.md` — add v0.1.4 footnote (parallel to existing v0.1.2 footnote)
- `docs/adrs/ADR-006-runtime-context-not-baked.md` — add v0.1.4 footnote (parallel to existing v0.1.2 footnote)
- `docs/adrs/ADR-008-opus-for-epic-reviewer-only.md` — add v0.1.4 footnote

NOT modified:
- `CHANGELOG.md` — historical entries v0.1.0–v0.1.3 preserved verbatim. The new v0.1.4 entry is added by S2.7.
- `docs/planning/prompts/ruckus-pm-agent-v0.1.2.md` — historical artifact, preserve filename and content
- `docs/planning/prompts/ruckus-v0.1.2-issues.md` — historical artifact
- `docs/planning/prompts/roughly-pm-agent-v0.1.4.md` — already correctly uses `roughly`
- `docs/planning/archive/**` — historical artifacts
- `docs/planning/epics/complete/**` — historical artifacts (E01)
- **`docs/plans/**`** — historical implementation plans from E01; preserve verbatim. (8 files, 6 contain "ruckus" references — these describe what was implemented under the old name; rewriting them would falsify the historical record.)
- ADR-001, ADR-002, ADR-003, ADR-007 — discovery confirmed no "ruckus" mentions
- **ADR body text in ADR-004/005/006/008** — body remains unchanged; only footnote is added. The inline `/ruckus:*` references in ADR-004 (L11, L17, L27) and ADR-005 (L17, L35) and ADR-006 are HISTORICAL DECISION TEXT — they describe the namespace as it was when the ADR was Accepted. The footnote provides the rename annotation. This preserves ADR immutability except for the explicitly-permitted footnote addition.

**Context:**

Active documentation files describe the plugin to users and contributors. After the rename, every prose reference becomes `Roughly` (sentence-start) or `roughly` (mid-sentence/identifier), every code-block `/ruckus:*` becomes `/roughly:*`, and clone-path references update from `~/.claude/plugins/ruckus/` to `~/.claude/plugins/roughly/` and from `/path/to/your/ruckus-clone` to `/path/to/your/roughly-clone` (matching the GitHub repo rename the human will execute after E02 ships).

ADRs received temporary edit permission to add v0.1.4 footnotes parallel to the existing v0.1.2 footnotes that ADR-005 (L52) and ADR-006 (L47) already carry. The v0.1.4 footnote pattern (matching the v0.1.2 footnote style):

```markdown
> **Note (v0.1.4):** The plugin was renamed from `ruckus` to `roughly`. Slash commands now use the `/roughly:*` namespace; the plugin-installed dotdir is `.roughly/`. Original identifiers above reflect the original naming.
```

**Footnote placement rules:**
- For **ADR-005 and ADR-006** (which already carry a v0.1.2 footnote): append the v0.1.4 footnote on the line immediately after the existing v0.1.2 footnote, separated by a single blank line. Both footnotes appear at the very end of the file, in chronological order (v0.1.2 first, v0.1.4 second). Do not insert any other content between them or after the v0.1.4 footnote.
- For **ADR-004 and ADR-008** (which currently have no footnote): append the v0.1.4 footnote at the end of the file, preceded by a single blank line after the last existing line. The footnote is the new last line of the file.
- All four footnotes use the exact text shown above.

ADR body text — including inline `/ruckus:*` references in decision and alternative-considered sections — is PRESERVED. The footnote is the only change to the ADR file. This minimizes the editing surface on Accepted documents and keeps the "decisions are immutable" convention intact.

**Migration discoverability — README "Migrating from ruckus to roughly" subsection:**

Add a new subsection to `README.md` titled `## Migrating from ruckus (v0.1.3) to roughly (v0.1.4)` (placement: directly after the `### Your first feature` subsection inside Quick Start — currently ending at L55 with the "8 gates" wrap-up sentence — and immediately before `## How It Works` at L57). Content:

```markdown
## Migrating from ruckus (v0.1.3) to roughly (v0.1.4)

If you were using the previous `ruckus` plugin, follow these steps once per machine and once per project:

1. **Install the new plugin under the `roughly` name:**
   ```
   /plugin marketplace add nickkirkes/roughly
   /plugin install roughly@nickkirkes
   ```
2. **Run `/roughly:upgrade` from each project that previously used `/ruckus:*`.** The upgrade skill detects the legacy `.ruckus/` directory, prompts you, and migrates known-pitfalls.md, workflow-upgrades, and any path references in your root CLAUDE.md to `.roughly/`.
3. **Optionally uninstall the old plugin:** `/plugin uninstall ruckus`. The new and old plugins can coexist temporarily, but only `/roughly:upgrade` runs the migration; the old `/ruckus:*` commands continue to operate on the legacy paths until uninstalled.

The rename is a hard cut — there are no aliases or backwards-compatibility shims. Behavior is identical to v0.1.3; only names, paths, and namespace identifiers change.
```

Add a parallel `### Migration` subsection inside the v0.1.4 CHANGELOG entry (handled in S2.7) that mirrors these three steps.

This story is documentation-only and does not modify behavior. It does not require 3-agent validation.

**`docs/planning/README.md` is updated by the PM directly** as part of the E02 deliverable write-up — it falls under the PM's standing authority per the PM prompt and is not assigned to an implementation agent here.

**Current behavior (excerpts):**

```markdown
// README.md L1
# Ruckus

// README.md L8 (install commands users copy-paste)
/plugin marketplace add nickkirkes/ruckus
/plugin install ruckus@nickkirkes

// CLAUDE.md L1
# Ruckus

// CONTRIBUTING.md L1
# Contributing to Ruckus
```

**Target behavior:** Replace `Ruckus` with `Roughly` in titles and prose; replace `ruckus` with `roughly` in CLI commands, install paths, clone paths. Append the v0.1.4 footnote to ADR-004, ADR-005, ADR-006, ADR-008.

**ADR constraints:**
- This story modifies four ADRs by addition only (footnote append). Decision text is preserved. The temporary edit permission was granted by the human for E02.

**Cross-reference impacts:**
- README.md mentions `.ruckus/` paths in its project structure tree — those references update from `.ruckus/` to `.roughly/` here (text-only; the actual directory rename is S2.6's territory).
- CLAUDE.md structure table references `.ruckus/` — text update here; the actual directory rename is S2.6's territory.

**Acceptance Criteria:**

- [x] `README.md` has `# Roughly` as title; opening "Roughly is a Claude Code plugin..."; install commands `/plugin marketplace add nickkirkes/roughly` and `/plugin install roughly@nickkirkes`; clone path `~/.claude/plugins/roughly/` at L12; 22 `/roughly:*` references in code blocks; zero `\bRuckus\b` capital-R prose; `/ruckus:*` matches limited to the "Migrating from ruckus" subsection prose at L66/L67 (allow-listed) (verified 2026-04-30)
- [x] `README.md` contains the new "Migrating from ruckus (v0.1.3) to roughly (v0.1.4)" subsection at L57 with the three-step user instructions: install new plugin, run `/roughly:upgrade` per project, optionally `/plugin uninstall ruckus` (verified 2026-04-30)
- [x] `CLAUDE.md` has `# Roughly`; opens with "Roughly is a Claude Code plugin..."; references `/roughly:setup` and `roughly-clone`; zero `ruckus` residue (verified 2026-04-30 — `rg -i ruckus CLAUDE.md` returns zero)
- [x] `CONTRIBUTING.md` has `# Contributing to Roughly`; clone paths use `roughly-clone` at L8 and L54; 4 `/roughly:*` references in CLI examples; zero `ruckus` residue (verified 2026-04-30)
- [x] ADR-004 has v0.1.4 footnote appended at end of file (verified 2026-04-30: footnote is the last paragraph; ADR body text unchanged)
- [x] ADR-005 has v0.1.4 footnote appended below the existing v0.1.2 footnote (verified 2026-04-30: both footnotes in chronological order with single blank line separator; body unchanged)
- [x] ADR-006 has v0.1.4 footnote appended below the existing v0.1.2 footnote (verified 2026-04-30: both footnotes in chronological order with single blank line separator; body unchanged)
- [x] ADR-008 has v0.1.4 footnote appended at end of file (verified 2026-04-30; body unchanged)
- [x] All four v0.1.4 footnotes use the exact canonical text (verified 2026-04-30 by literal string match)
- [x] ADR-001, ADR-002, ADR-003, ADR-007 are unchanged (verified 2026-04-30: zero `Note (v0.1.4)` matches; not in branch diff)
- [x] Historical CHANGELOG entries v0.1.0–v0.1.3 are unchanged (verified 2026-04-30: CHANGELOG.md is not modified in PR #21's branch diff)
- [x] `docs/plans/**` historical files are unchanged (verified 2026-04-30: branch diff shows only the NEW `S02.5-documentation-plan.md` — appropriate per the build-pipeline plan-creation flow — no edits to existing E01 plans)

**Post-merge audit (2026-04-30):**

S2.5 PR #21 review-passed cleanly. Three commits on the branch: `0ae8511` (main rename + ADR footnotes), `4ff8e5a` (two pitfalls captured during S2.5 wrap-up — on-pattern per Stage 8 / ADR-006), `7b6ad8f` (repo-relative paths in plan verify commands — small docs cleanup discovered during the rename pass). 9 files changed: README.md (+49 net lines, mostly the Migration subsection), CLAUDE.md (10 changes), CONTRIBUTING.md (14 changes), 4 ADRs (+2 lines each for footnotes), `.roughly/known-pitfalls.md` (+4 lines for new pitfalls), and the new `docs/plans/S02.5-documentation-plan.md` (194-line implementation plan, on-pattern artifact). All five reviewer-flagged scope items (footnote placement rules, README anchor, ADR body preservation, no `~roughly` brand wordmark in repo, canonical footnote text) verified compliant. No scope expansions of significance — the pitfalls capture and plan-paths fix are routine.

---

### S2.6: This repo's own `.ruckus/` → `.roughly/` dogfood ✅

**Status:** Complete — PR #20 review passed 2026-04-30 (commit `3cff90a`). Merge pending.
**Priority:** P2 (parallelizable with S2.2/S2.3/S2.4/S2.5 after S2.1; trivial; not validated)

**Files:**
- `.ruckus/known-pitfalls.md` — `git mv` to `.roughly/known-pitfalls.md` AND content updates at L3, L4, L6, L12
- `.ruckus/workflow-upgrades` — `git mv` to `.roughly/workflow-upgrades` AND content update at L1

**Context:**

This repo dogfoods itself: the Ruckus plugin uses Ruckus on its own development. The `.ruckus/` directory at the repo root contains the same two files the plugin installs into user projects. The rename moves both to `.roughly/`. These files are not shipped to users (templates in S2.4 ship to users; this is dogfood).

**Current state per discovery:**
- `.ruckus/known-pitfalls.md` L3: `Project: Ruckus`
- `.ruckus/known-pitfalls.md` L4: `Domain: Ruckus is a Claude Code plugin that implements gated development pipelines with subagent-per-task execution.`
- `.ruckus/known-pitfalls.md` L6: `Pitfalls discovered through development. Updated by /ruckus:build and /ruckus:fix wrap-up stages.`
- `.ruckus/known-pitfalls.md` L12: `subagent_type` example string `ruckus:code-reviewer`
- `.ruckus/workflow-upgrades` L1: `ruckus-version 0.1.3 2026-04-28`

**Target behavior:**
- `git mv .ruckus .roughly`
- `.roughly/known-pitfalls.md` L3: `Project: Roughly`
- L4: `Domain: Roughly is a Claude Code plugin...`
- L6: `Updated by /roughly:build and /roughly:fix...`
- L12: `roughly:code-reviewer`
- `.roughly/workflow-upgrades` L1: `roughly-version 0.1.3 2026-04-28` (version digits and date preserved; STEP 6 of upgrade later updates to 0.1.4 if `/roughly:upgrade` is run on this repo, but for now S2.7 will set the version directly via the plugin.json bump)

**ADR constraints:** None — this is project state, not plugin source.

**Cross-reference impacts:** None. Self-contained.

**Acceptance Criteria:**
- [x] `.ruckus/` directory no longer exists in the repo (verified 2026-04-30: `ls -la .ruckus` returns "No such file or directory")
- [x] `.roughly/known-pitfalls.md` exists with content updated per the four lines (verified 2026-04-30: L3 `Project: Roughly`, L4 `Domain: Roughly is a Claude Code plugin...`, L6 `Updated by /roughly:build and /roughly:fix...`, L12 `roughly:code-reviewer` in subagent_type example)
- [x] `.roughly/workflow-upgrades` exists with `roughly-version 0.1.3 2026-04-28` (verified 2026-04-30: `sed -n '1p' .roughly/workflow-upgrades` returns the expected line)
- [x] `git log --follow .roughly/known-pitfalls.md` shows continuity from `.ruckus/known-pitfalls.md` (verified 2026-04-30: log chain runs through `3cff90a` (S2.6 rename) → `f1bb2f1` (S2.3 pitfalls capture) → `536100f` (S2.2) → `cea271f` (E01 audit) → `1807c2d` (the prior `docs/claude/` → `.ruckus/` migration). Two-tier rename history preserved across both v0.1.2 and v0.1.4 transitions.)

---

### S2.7: Final verification, version bump, CHANGELOG, tag

**Status:** Not Started
**Priority:** P0 (last; never parallelized)

**Files:**
- `.claude-plugin/plugin.json` — version bump to `0.1.4`
- `CHANGELOG.md` — new v0.1.4 entry

**Context:**

After all prior stories merge, this story performs final verification, version bump, CHANGELOG, and tag. Only this story touches `.claude-plugin/plugin.json` for `version` (S2.1 touched it for `name`). It must not run in parallel with any other story.

**Verification steps:**

**Audit timing:** Run the static residue audit (step 1) AFTER S2.6 has merged. S2.6 renames the dogfood `.ruckus/` → `.roughly/`; running the audit before that rename produces false-positive matches on the directory name itself. The recommended sequence is: all stories merged → S2.6 dogfood rename verified on disk → run the audit.

1. **Static residue audit:** Run `rg -i ruckus` from repo root. The only ALLOWED remaining matches are:
    - Historical CHANGELOG entries v0.1.0–v0.1.3 (`CHANGELOG.md`)
    - `docs/planning/archive/**` (historical artifacts)
    - `docs/planning/epics/complete/**` (E01 epic and audit)
    - `docs/planning/prompts/ruckus-pm-agent-v0.1.2.md` and `docs/planning/prompts/ruckus-v0.1.2-issues.md` (historical prompts)
    - `docs/plans/**` (historical E01 implementation plans, 6 of 8 files contain matches)
    - Migration-context references INSIDE the v0.1.4 migration step in `skills/upgrade/SKILL.md` (intentionally name `.ruckus/` to detect legacy installations)
    - Migration-prompt prose INSIDE the v0.1.2 migration step in `skills/upgrade/SKILL.md` (the prose narration of the prior rename)
    - The v0.1.4 footnotes in ADR-004/005/006/008 (mention "ruckus" as the prior name) and the existing v0.1.2 footnotes in ADR-005/006
    - **Capital-R `Ruckus` prose in ADR body text:** ADR-005 L11, ADR-006 L11, ADR-008 L11 (only these three lines) — preserved as historical decision text per S2.5
    - **Inline `/ruckus:*` namespace tokens in ADR body text:** ADR-004 L11/L17/L27, ADR-005 L17/L35 — preserved as historical decision text per S2.5
    - **Lowercase `.ruckus/` path references in ADR body text:** ADR-005 L17, ADR-006 L17 — preserved as historical decision text per S2.5
    - This E02 epic file at `docs/planning/epics/E02-rename-roughly.md` (intentionally describes the rename)
    - The v0.1.4 PM prompt file at `docs/planning/prompts/roughly-pm-agent-v0.1.4.md` (describes the rename)
    - The new "Migrating from ruckus (v0.1.3) to roughly (v0.1.4)" subsection in `README.md` and the parallel `### Migration` subsection inside the v0.1.4 entry of `CHANGELOG.md` (added by S2.5 and S2.7 respectively)
    - This repo's git history (logs and old commit messages — not part of the file system audit)
2. **Plugin loads:** Manual test from a fresh project: `claude --plugin-dir <ruckus repo>`. Confirm `/roughly:setup`, `/roughly:build`, `/roughly:review`, `/roughly:verify-all`, `/roughly:fix`, `/roughly:upgrade`, `/roughly:review-plan`, `/roughly:review-epic`, `/roughly:audit-epic` all resolve.
3. **Cross-reference integrity:** Every `subagent_type` in skills resolves; every cross-skill reference uses `/roughly:*` and resolves.
4. **End-to-end pipeline test:** Run `/roughly:setup` in a scratch directory, then `/roughly:build` against a small test plan. Verify the new namespace dispatches; verify `.roughly/` directory is created (not `.ruckus/`); verify `roughly-version` line in `.roughly/workflow-upgrades`.
5. **v0.1.4 upgrade migration test:** In a scratch directory, manually create `.ruckus/known-pitfalls.md` (containing `/ruckus:build` and `/ruckus:fix` boilerplate at L6) and `.ruckus/workflow-upgrades` (containing `ruckus-version 0.1.3 2026-04-28`). Run `/roughly:upgrade`. Verify the v0.1.4 migration step prompts and:
    - Writes `.ruckus/.migration-in-progress` marker before any moves; removes it on success
    - Moves files to `.roughly/`
    - Renames `ruckus-version` → `roughly-version` in workflow-upgrades content (preserving the date)
    - Rewrites `/ruckus:build` and `/ruckus:fix` → `/roughly:*` in the boilerplate L6 of known-pitfalls.md
    - Displays before/after match counts when updating root CLAUDE.md path references
    - On a re-run after a successful migration, finds no `.ruckus/` and skips the step entirely
    - Removes empty `.ruckus/`
5b. **Modified v0.1.2 step regression test (NEW):** In a separate scratch directory, manually create only `docs/claude/known-pitfalls.md` and `docs/claude/.workflow-upgrades` (simulating a v0.1.0/v0.1.1 user upgrading directly to v0.1.4 without ever installing v0.1.2). Run `/roughly:upgrade`. Verify the modified v0.1.2 step migrates straight to `.roughly/` (no `.ruckus/` intermediate is created at any point) and the post-migration root CLAUDE.md references `.roughly/`.
6. **Conflict-case test (no marker):** In a scratch directory with both `.ruckus/` and `.roughly/` populated AND no `.ruckus/.migration-in-progress` marker, run `/roughly:upgrade`. Verify the conflict prompt fires and offers `diff / keep .roughly / keep .ruckus / abort`. On `abort`, verify the upgrade stops with the expected message and the marker is NOT written.
6b. **Partial-failure resume test (NEW):** In a scratch directory, simulate a half-migrated state: create `.roughly/known-pitfalls.md`, `.ruckus/workflow-upgrades`, AND a `.ruckus/.migration-in-progress` marker. Run `/roughly:upgrade`. Verify the v0.1.4 step detects the marker, classifies as a half-completed migration, skips the conflict prompt, and resumes from step 5 (move) — completing the partial migration without prompting.
7. **Partial-content rewrite verification:** When the v0.1.4 migration encounters a customized boilerplate line in known-pitfalls.md (the regex doesn't match), verify the upgrade summary contains the warning: `"Custom boilerplate line in .roughly/known-pitfalls.md preserved verbatim — check for stale /ruckus:* tokens manually."`

**Version bump:** `.claude-plugin/plugin.json` `"version": "0.1.3"` → `"version": "0.1.4"`.

**CHANGELOG entry:**

```markdown
## [0.1.4] — [date]

### Changed
- Renamed plugin from "ruckus" to "roughly". Hard cut with no aliases or backwards compatibility.
- Slash command namespace migrated from `/ruckus:*` to `/roughly:*`.
- Plugin-installed dotdir migrated from `.ruckus/` to `.roughly/`.
- Workflow-upgrades version-line identifier renamed from `ruckus-version` to `roughly-version`.

### Added
- Upgrade migration step (v0.1.4): detects `.ruckus/` in existing projects, prompts the user, moves files to `.roughly/`, renames the version-line identifier in workflow-upgrades, rewrites the `/ruckus:build` and `/ruckus:fix` boilerplate inside known-pitfalls.md, updates root CLAUDE.md path references with before/after counts. Includes an in-progress marker file for partial-failure recovery, an interactive prompt for user-extra files in `.ruckus/`, and a conflict-handling prompt for the both-dirs-exist case.
- v0.1.4 footnotes on ADR-004, ADR-005, ADR-006, ADR-008 documenting the namespace and dotdir change. ADR decisions and body text are unchanged; only footnotes are added.

### Migration
If you were using the previous `ruckus` plugin, follow these steps once per machine and once per project:
1. Install the new plugin under the `roughly` name: `/plugin marketplace add nickkirkes/roughly` followed by `/plugin install roughly@nickkirkes`.
2. Run `/roughly:upgrade` from each project that previously used `/ruckus:*`. The upgrade detects the legacy `.ruckus/` directory and migrates known-pitfalls.md, workflow-upgrades, and any path references in your root CLAUDE.md to `.roughly/`.
3. Optionally uninstall the old plugin: `/plugin uninstall ruckus`. The new and old plugins can coexist temporarily, but only `/roughly:upgrade` runs the migration; the old `/ruckus:*` commands continue to operate on the legacy paths until uninstalled.

### Notes
- Behavior is identical to v0.1.3. Only names, paths, namespace identifiers, and the workflow-upgrades version-line identifier change.
- Prior CHANGELOG entries (v0.1.0 through v0.1.3) retain "ruckus" naming as historical fact.
- `docs/plans/**` historical implementation plans retain their original naming as historical fact.
- GitHub repo rename and external-surface updates (social preview, repo description) are out of scope for this release; tracked in the maintainer's external migration checklist.
```

**`### Migration` section is deliberately non-standard.** Keep-a-Changelog convention uses Added/Changed/Deprecated/Removed/Fixed/Security only. The `### Migration` section is a deliberate divergence for this release because the rename is the one and only release where existing users must take explicit action — the section's prominence outweighs strict convention conformance. The project's CHANGELOG is manually maintained with no automated tooling that consumes section names, so the divergence is safe. Implementation agents should preserve the `### Migration` heading as written; do NOT relocate the content under `### Notes` or `### Changed` for "Keep-a-Changelog cleanup."

**Tag:** After CHANGELOG entry committed, create tag `v0.1.4` locally on the verified HEAD. **No commits land between tag-create and tag-push.** If a hotfix is needed before the tag is pushed (e.g., a broken AC discovered during manual testing), delete the local tag, commit the fix, and re-create the tag on the new HEAD. Push is deferred per human's preference; the human will rename the GitHub repo first, then push the tag and any final adjustments. Nothing in `plugin.json` or `marketplace.json` embeds the GitHub repo URL, so the tag content is repo-rename-agnostic.

**Acceptance Criteria:**

- [ ] **Audit timing satisfied:** S2.6's dogfood `.ruckus/` → `.roughly/` rename has merged before this story's audit runs (verified: `ls -la` shows `.roughly/` and no `.ruckus/` at repo root)
- [ ] `rg -i ruckus` from repo root returns ONLY the allow-listed historical, migration-context, footnote, and ADR-body-text matches enumerated above (zero unintended residue)
- [ ] `.claude-plugin/plugin.json` `version` is `"0.1.4"`
- [ ] `CHANGELOG.md` has v0.1.4 entry with the `### Migration` subsection per the format above
- [ ] Plugin loads in a fresh Claude Code session
- [ ] All 9 slash commands resolve under the new namespace (listed above)
- [ ] No `/roughly:build-ui` pattern exists in any file (ADR-004 compliance — verified via `rg -n '/roughly:build-ui'` returning zero)
- [ ] End-to-end pipeline test (step 4) passes on the renamed repo
- [ ] **v0.1.4 upgrade migration test (step 5) passes**: marker is written/removed, files moved, version-line identifier renamed, boilerplate rewritten, CLAUDE.md path references updated with displayed counts, idempotent re-run skips entirely
- [ ] **v0.1.2 step regression test (step 5b) passes** (NEW): a `docs/claude/`-only scratch dir migrates straight to `.roughly/` with no `.ruckus/` intermediate
- [ ] Conflict-case test (step 6) passes: prompt fires when both dirs exist with no marker; abort stops cleanly
- [ ] **Partial-failure resume test (step 6b) passes** (NEW): marker present + both dirs → resume from step 5 without prompting
- [ ] Customized-boilerplate warning test (step 7) passes: warning appears in upgrade summary when regex doesn't match
- [ ] Tag `v0.1.4` exists locally on the verified HEAD; no commits between tag-create and tag-push window

---

## Exit Criteria

- [ ] All 7 stories complete (S2.1–S2.7)
- [ ] `rg -i ruckus` returns zero unintended residue (only allow-listed)
- [ ] No `/roughly:build-ui` pattern exists (ADR-004 compliance)
- [ ] Plugin loads, all skills/agents/commands resolve under the new namespace
- [ ] `/roughly:review` and `/roughly:verify-all` execute cleanly
- [ ] `CLAUDE.md` updated to reflect the new name (S2.5)
- [ ] `README.md` updated with new name (S2.5)
- [ ] `CHANGELOG.md` updated with v0.1.4 entry (S2.7)
- [ ] All cross-references valid (skills reference existing agents under new names, agents reference existing files under new paths)
- [ ] `plugin.json` `name` = `roughly` (S2.1) AND `version` = `0.1.4` (S2.7)
- [ ] `marketplace.json` updated with new name (S2.1)
- [ ] Tag `v0.1.4` created locally
- [ ] Discovery audit re-run after final commit confirms zero unintended residue (S2.7)

---

## Parallelization Notes

**Sequence:**

| Phase | Stories | Rationale |
|---|---|---|
| 1 | S2.1 | Foundation — plugin identity and namespace must settle first |
| 2 | S2.2 | Skills paths/version-line/prose/migration step. **Recommend serial after S2.1** because S2.1 and S2.2 share files (build/SKILL.md, fix/SKILL.md, setup/SKILL.md, upgrade/SKILL.md) on disjoint lines but with mixed-content lines (build L250, fix L255, setup L40) that cause textual conflicts under parallel merging. |
| 3 | S2.3, S2.4, S2.5, S2.6 | Disjoint file sets — fully parallelizable in worktrees if the human wants velocity |
| 4 | S2.7 | Verification, version bump, CHANGELOG, tag — must be last and serial |

**Disjoint file sets in Phase 3:**
- S2.3 owns: `agents/*` (excluding static-analysis.md)
- S2.4 owns: `skills/setup/templates/*`
- S2.5 owns: `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `docs/adrs/ADR-{004,005,006,008}*.md`
- S2.6 owns: `.ruckus/*` (renamed to `.roughly/*`)

**Recommendation:** Sequential S2.1 → S2.2 → (S2.3 ‖ S2.4 ‖ S2.5 ‖ S2.6) → S2.7 is the safest path. Pure-sequential is also fine if the human prefers single-branch simplicity.

**Implicit S2.5 → S2.6 dependency:** S2.5 updates `README.md` and `CLAUDE.md` text to reference `.roughly/` paths in the project structure tree and structure table. S2.6 actually performs the dogfood `.ruckus/` → `.roughly/` rename in the repo. If S2.5 lands in Phase 3 before S2.6 also lands, the README and CLAUDE.md briefly say `.roughly/` while the dogfood is still `.ruckus/` — an acceptable transient inconsistency that resolves the moment S2.6 merges. **Recommend running S2.6 before S2.5 in Phase 3 if both are queued in parallel**, so the docs and the on-disk reality stay in sync. S2.7's audit-timing requirement (audit runs after S2.6) makes this a hard constraint at the end of Phase 3 anyway.

---

## Validation Summary

### v1 (initial draft) — 2026-04-28

12 validation agents (3 per flagged story × 4 flagged stories: S2.1, S2.2, S2.3, S2.4) ran in parallel on 2026-04-28. Issues identified and resolved:

- S2.1: removed duplicate fix/SKILL.md L56 entry; moved `no /roughly:build-ui` AC to Exit Criteria; clarified mixed-content-line coordination with S2.2
- S2.2: added 5th `ruckus-version` reference (setup L132); added skill-prose `Ruckus` references (build L250, fix L255, setup L40); specified v0.1.2 substring rewrite explicitly; specified v0.1.4 migration step in 8 numbered points covering edge cases (`.roughly/` already exists, both legacy dirs exist, content rewrites for version-line identifier and known-pitfalls.md boilerplate, idempotency); added line budget verification
- S2.3: replaced byte-for-byte sync AC with concrete path-string check; added agent-preamble.md L6–L8 header-comment update for doc-writer
- S2.4: added cross-reference notes pointing to S2.2 for migration content-rewrite responsibility
- S2.5: explicit ADR scope (footnote only, body preserved); added `docs/plans/**` to NOT-modified list
- S2.7: added `docs/plans/**` to allow-list; explicit conflict-case and content-rewrite verification tests; absorbed `no /roughly:build-ui` AC

ADR/Naming reviewers reported all four flagged stories clean on ADR compliance. Cross-Reference reviewers identified the issues now incorporated above.

### v2 (post `/ruckus:review-epic`) — 2026-04-28

The Opus epic-reviewer (`/ruckus:review-epic`) ran on the v1 draft and returned `Needs Revision` with 10 prioritized recommendations. All addressed in this v2:

- **S2.3 line-citation fix**: corrected from "L6–L8 header comment" to "the HTML comment block at L3–L13" with explicit `<!--`/`-->` delimiter-preservation requirement; added a render-validation AC.
- **S2.3 doc-writer placement (Q3 outcome)**: kept the L5–L8 manual-sync target list unchanged (5 inliners only); appended a doc-writer path-sync sentence to the L9–L13 exceptions paragraph instead of growing the sync list.
- **S2.2 spec expanded from 8 to 10 numbered points** covering: (1) git-vs-plain-mv detection method, (2) marker file `.ruckus/.migration-in-progress` for partial-failure idempotency, (3) conflict-check-OR-resume branching with explicit `abort` behavior, (5) idempotent move semantics, (6) anchored regex on boilerplate rewrite + warn-on-skip, (7) literal-substring CLAUDE.md update with displayed match counts, (8) interactive prompt for user-extra files (Q2 outcome), (9) marker cleanup, (10) idempotency contract.
- **S2.2 in-place character substitution constraint (Q1 outcome)**: build/SKILL.md and fix/SKILL.md must end at exactly 294 and 299 lines respectively — character-substitutions only, no line additions or removals. upgrade/SKILL.md exempt because it gains the new 10-point step.
- **S2.2 mixed-content line guidance**: explicit instruction to S2.1 ("edit ONLY the namespace token; leave prose `Ruckus` for S2.2") and S2.2 ("no-op if S2.1 already replaced both").
- **S2.2 grep ACs tightened**: greps are run-and-audit-against-allow-list, not pure zero-match assertions; allow-list explicitly enumerates v0.1.2 and v0.1.4 migration prompt text.
- **S2.5 footnote placement**: explicit rules — ADR-005/006 get v0.1.4 footnote appended below the existing v0.1.2 footnote (one blank line separator); ADR-004/008 get the footnote as the new last line preceded by one blank line. All four use the canonical footnote text.
- **S2.5 migration discoverability**: README gains a "Migrating from ruckus (v0.1.3) to roughly (v0.1.4)" subsection with three-step user instructions; CHANGELOG v0.1.4 entry gains a parallel `### Migration` subsection.
- **S2.7 allow-list additions**: capital-R `Ruckus` prose in ADR body text (ADR-004/005/006/008); v0.1.2 migration prompt prose in upgrade/SKILL.md; the new README and CHANGELOG migration subsections.
- **S2.7 audit timing**: explicit requirement that the residue audit runs AFTER S2.6's dogfood rename merges.
- **S2.7 step 5b regression test**: NEW — `docs/claude/`-only scratch dir migrates straight to `.roughly/` (no `.ruckus/` intermediate), validating the modified v0.1.2 step.
- **S2.7 step 6b partial-failure resume test**: NEW — marker + both dirs → step resumes from move without prompting.
- **S2.7 step 7 customized-boilerplate warning test**: NEW — verifies the warning text in upgrade summary.
- **S2.7 tag-timing note**: "no commits between tag-create and tag-push"; if hotfix needed, delete and recreate tag.
- **Parallelization Notes**: documented the implicit S2.5 → S2.6 dependency for transient consistency.

**Reviewer concerns NOT addressed (with rationale):**
- *Reviewer minor: `subagent_type.*ruckus` regex looseness in S2.1.* The looser pattern (without trailing `:`) is intentionally conservative — it catches both `subagent_type: "ruckus:..."` and any inadvertent prose like `# ruckus subagent_type` on the same line. Keeping looser.
- *Reviewer note: docs/planning/** gitignored.* Acknowledged as out-of-scope for E02; flagged in the planning README's deferred-items section for the maintainer to consider after the rename ships.

Re-running `/ruckus:review-epic` on this v2 is recommended before starting S2.1, but the precision/completeness nature of every revision means no architectural shape changed — only spec detail and AC coverage.

### v2 second Opus pass — 2026-04-28

After v2 was drafted, a second Opus `/ruckus:review-epic` pass returned `Ready` with three narrow corrections (saved in `E02-rename-roughly-review-v2.md` alongside this epic). All three were applied in-place and are now reflected above:

- **S2.7 allow-list line-citation drift:** the original v2 conflated capital-R `Ruckus` prose with lowercase `/ruckus:*` and `.ruckus/` tokens. Replaced the single-bullet allow-list at L565 with three accurate bullets: capital-R prose (ADR-005/006/008 L11 only), inline `/ruckus:*` namespace tokens (ADR-004 L11/L17/L27 + ADR-005 L17/L35), and lowercase `.ruckus/` path references (ADR-005 L17, ADR-006 L17).
- **S2.2 marker abort wording:** clarified L172 to distinguish the two abort behaviors — conflict-prompt path (no marker exists, re-running re-prompts) vs. partial-failure path (marker present, re-running resumes from step 5 without re-prompting).
- **S2.5 README placement anchor:** replaced the loose 70-line "after install/quick-start, before Skills Reference" zone with a single anchor — directly after `### Your first feature` (ending L55) and immediately before `## How It Works` (L57).

The remaining low-severity flags from the second pass have all been applied in this revision (no remaining deferral):

- **S2.2 read-only-fs marker write failure (v2 issue #4):** point 2 of the 10-point spec now specifies the abort behavior with the exact error message when the marker write fails on a read-only filesystem or restrictive permissions. No partial state is created.
- **S2.2 marker cleanup ordering when user picks `leave` for extras (v2 issue #7):** point 9 of the 10-point spec now mandates removing the marker file regardless of whether `.ruckus/` is also removed, with explicit rationale (a stale marker would trigger a false partial-failure-resume on the next run).
- **S2.7 non-standard `### Migration` CHANGELOG section (v2 issue #5):** retained as-is per a deliberate human-readability choice. A new note in the S2.7 spec, just before the Tag heading, documents this divergence so an implementation agent does not "fix" it back to Keep-a-Changelog convention.

Only one v2 reviewer flag is intentionally NOT addressed: the stylistic note on S2.7 AC L621 wording (v2 issue #6) — the reviewer explicitly recommended "leave as-is" because the parenthetical clarification makes the AC observable post-hoc.

---

## References

- [CLAUDE.md](../../../CLAUDE.md) — project conventions
- [ADR-001 through ADR-008](../../adrs/) — design decisions (preserved unchanged in spirit; ADR-004/005/006/008 receive v0.1.4 footnotes per S2.5)
- [E01 epic](complete/E01-directory-rename-pipeline-hardening.md) — prior precedent for a directory-rename release
- [E01 audit](complete/E01-directory-rename-pipeline-hardening-audit.md) — audit format reference
- [v0.1.4 PM prompt](../prompts/roughly-pm-agent-v0.1.4.md) — source of truth for this rename
- Discovery findings (inlined per-story in Context sections; consolidated in the Discovery Inventory at the top of this epic)
