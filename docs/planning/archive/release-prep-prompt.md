# Task: Documentation Audit & v0.1.0 Release Prep

## Context

Ruckus is a Claude Code plugin by Rowdy Cloud that implements gated development pipelines with subagent-per-task execution. It has been through 5 phases of review + a post-Phase 5 review with all critical and regression issues resolved. The codebase is on `main` and ready for tagging as v0.1.0.

Before tagging, the documentation layer needs to be complete and accurate. This is an open-source project — the repo IS the first impression. A developer who clones it should understand what they're looking at, how to use it, how to contribute, and how to debug it, within 15 minutes of reading.

## Step 1: Audit Current State

Read everything before changing anything. Map what exists, what's missing, and what's stale.

```bash
# What docs exist
find . -name "*.md" -not -path "./node_modules/*" -not -path "./.git/*" | sort

# README size and structure
wc -l README.md
grep "^#" README.md

# Check for CLAUDE.md
ls -la CLAUDE.md 2>/dev/null || echo "No CLAUDE.md"

# Check for ADRs
ls -la adrs/ 2>/dev/null || echo "No adrs/ directory"

# Check for contributing guide
ls -la CONTRIBUTING.md 2>/dev/null || echo "No CONTRIBUTING.md"

# Check plugin.json version
cat .claude-plugin/plugin.json

# Check CHANGELOG
cat CHANGELOG.md 2>/dev/null || echo "No CHANGELOG.md"

# Check LICENSE
cat LICENSE 2>/dev/null || echo "No LICENSE"

# Skill and agent inventory
echo "=== Skills ==="
for f in skills/*/SKILL.md; do echo "$f: $(head -5 $f | grep 'name:')"; done

echo "=== Agents ==="
for f in agents/*.md; do echo "$f: $(head -5 $f | grep 'name:')"; done

# Git state
git log --oneline -20
git tag -l
```

Report what you find. Then proceed through the following steps.

## Step 2: Create or Update CLAUDE.md

This project is a Claude Code plugin — it should absolutely have its own CLAUDE.md. This file is read by any Claude Code session working on the repo (including contributors using Claude Code to develop Ruckus itself).

CLAUDE.md for Ruckus should include:

**Project Identity:**
- What Ruckus is (one sentence)
- That it's a Claude Code plugin, not a standalone app
- MIT licensed, by Rowdy Cloud

**Structure:**
- Where skills live (`skills/<name>/SKILL.md`)
- Where agents live (`agents/<name>.md`)
- Where templates live (`skills/setup/templates/`)
- Where ADRs live (`adrs/`)
- What `agent-preamble.md` is and why it exists
- What `spec-reviewer-prompt.md` is and why it's shared (reference ADR-003)

**Build/Test:**
- There is no build step — this is pure markdown
- How to test locally: `claude --plugin-dir ./` in a test project
- How to verify structure: check frontmatter, cross-references, file counts

**Conventions:**
- Skills must have YAML frontmatter with `name` and `description`
- Agents must have YAML frontmatter with `name`, `description`, `tools`, `model`
- Pipeline skills (build, fix) must have `disable-model-invocation: true`
- Coordinator skills (review, verify-all, audit-epic, review-epic) must have `disable-model-invocation: true`
- Agent prompts must stay under 500 words
- Skill bodies must stay under 300 lines
- Project-specific values use `{{PLACEHOLDER}}` markers — never hardcode project names, commands, or paths
- Maturity check IDs must be versioned (e.g., `investigator-v1`)
- All changes need ADRs if they meet the bar defined in `adrs/README.md`

**Key Design Decisions (reference, not duplicate):**
- Point to `adrs/` directory for full reasoning
- One-line summary of each ADR for quick orientation

**Known Pitfalls for Contributors:**
- Skill content runs in the USER's project context, not the plugin directory — file paths must resolve relative to the user's repo, not the plugin source
- `disable-model-invocation: true` is critical for coordinator skills — without it, Claude may answer conversationally instead of dispatching agents
- Avoid "human override" language in gates — LLMs interpret ambiguous override permissions as license to skip
- Template conditionals (`{{#IF_UI_TASK}}`) need explicit evaluation instructions — the orchestrator doesn't have a template engine

Keep CLAUDE.md under 150 lines. Reference other docs rather than inlining their content.

## Step 3: Verify and Update README.md

Read README.md end to end. Check every claim against the actual codebase.

**Accuracy checks:**
- Does the skill count match the actual number of skills? Count `skills/*/SKILL.md`
- Does the agent count match? Count `agents/*.md` (excluding `agent-preamble.md`)
- Does the project structure tree match the actual directory structure? Run `find . -not -path './.git/*' -not -path './node_modules/*' -not -path './specs/*'` and compare
- Does the installation command work? (Verify the marketplace name matches `marketplace.json`)
- Does the Quick Start accurately describe what `/ruckus:setup` creates? Compare against `skills/setup/SKILL.md`
- Does the workflow decision matrix cover all 9 skills?
- Does the Skills Reference section describe every skill?
- Does the Agents section describe every agent?
- Does the token usage table reflect the Phase 4 optimizations (compaction boundaries)?
- Does the troubleshooting section reference accurate file locations?

**Completeness checks:**
- Is there a "What setup creates" list showing every file setup produces?
- Is `.workflow-upgrades` file location documented?
- Is the self-upgrading behavior documented with file location and decline handling?
- Is the relationship between `/ruckus:review` and `/ruckus:review-plan` explained?
- Are "subagent" and "orchestrator" defined before first use?
- Is there a "first feature" walkthrough (install → setup → build a small feature)?

**Fix anything that's wrong or missing.** Don't add fluff — keep it tight and accurate.

## Step 4: Create or Verify ADRs

Check if the `adrs/` directory exists with a README and ADR files. If ADRs are being added fresh:

1. Create `adrs/README.md` explaining what ADRs are, the numbering convention, and the status vocabulary
2. Add the 8 ADRs (provided separately or already in the repo):
   - ADR-001: Verify-plan dispatched as subagent
   - ADR-002: Subagent-per-task implementation
   - ADR-003: Shared spec-reviewer across build and fix
   - ADR-004: UI conditional, not forked command
   - ADR-005: Versioned maturity check IDs
   - ADR-006: CLAUDE.md as runtime context
   - ADR-007: Two-stage review kept for all tasks
   - ADR-008: Opus for epic-reviewer only

If ADRs already exist, verify they're accurate against the current codebase. Phase 1-5 changes may have altered behavior described in ADRs.

## Step 5: Create or Update CONTRIBUTING.md

This should cover:

**Getting Started:**
- Fork and clone
- Install locally: `claude --plugin-dir ./`
- Test in a scratch project

**What to Contribute:**
- Bug reports with reproduction steps
- Documentation improvements
- New agent definitions (with ADR if they change the pipeline)
- Template improvements

**What NOT to Contribute (without discussion first):**
- Changes to pipeline stage structure (covered by ADRs)
- New pipeline commands (build, fix are intentionally the only two)
- Changes to the subagent dispatch model (ADR-001, ADR-002)
- Removing `disable-model-invocation: true` from coordinator skills

**PR Process:**
- Fork → branch → PR against `main`
- PRs should include updated CHANGELOG entry
- If the change involves a design decision, draft an ADR

**Code Standards:**
- Reference CLAUDE.md conventions
- Skill bodies under 300 lines
- Agent prompts under 500 words
- Versioned maturity check IDs

## Step 6: Update CHANGELOG.md

Ensure the CHANGELOG accurately reflects ALL changes from Phases 1-5 and the post-Phase 5 fixes. Structure as:

```markdown
# Changelog

## [0.1.0] - 2026-04-22

### Added
- [list everything that was built]

### Fixed
- [list everything from Phase 1-5 reviews that was fixed]

### Changed
- [list behavioral changes from debates and reviews]
```

The CHANGELOG should be a complete record. A contributor reading it should understand every deliberate decision, not just "initial release."

## Step 7: Verify plugin.json Version

```bash
cat .claude-plugin/plugin.json
```

Ensure version is `0.1.0`. If it says `0.0.1` or anything else, update it.

## Step 8: Final Cross-Reference Check

Run a final consistency check:

```bash
# Every skill referenced in README exists
grep -oP '/ruckus:\w+[-\w]*' README.md | sort -u | while read cmd; do
  skill=$(echo $cmd | sed 's|/ruckus:||')
  if [ ! -f "skills/$skill/SKILL.md" ]; then
    echo "MISSING: $cmd references skill that doesn't exist at skills/$skill/SKILL.md"
  fi
done

# Every agent referenced in skills exists
grep -rh 'dispatch.*agent\|the.*agent' skills/ | grep -oP '`\w[-\w]*`' | sort -u | while read agent; do
  name=$(echo $agent | tr -d '`')
  if [ ! -f "agents/$name.md" ]; then
    echo "CHECK: $agent referenced in skills but no agents/$name.md found"
  fi
done

# File counts match README claims
echo "Skills: $(ls skills/*/SKILL.md 2>/dev/null | wc -l)"
echo "Agents: $(ls agents/*.md 2>/dev/null | grep -v preamble | wc -l)"

# No unreplaced placeholders in non-template files
grep -r '{{' skills/ agents/ --include="*.md" | grep -v template | grep -v 'IF_UI_TASK\|PLACEHOLDER explanation'
```

Report any mismatches. Fix them.

## Step 9: Tag Readiness Report

After all changes, produce a final report:

```
# v0.1.0 Tag Readiness

## Files Created/Updated
- [ ] CLAUDE.md — [created/updated]
- [ ] README.md — [verified accurate / updated]
- [ ] CONTRIBUTING.md — [created/updated]
- [ ] CHANGELOG.md — [verified complete / updated]
- [ ] adrs/ — [8 ADRs present and accurate]
- [ ] plugin.json — version is 0.1.0
- [ ] LICENSE — MIT, present

## Cross-Reference Status
- [ ] All skills referenced in README exist
- [ ] All agents referenced in skills exist
- [ ] File counts in README match actual counts
- [ ] No unreplaced placeholders outside templates

## Documentation Coverage
- [ ] New user can install and run setup from README alone
- [ ] Contributor can fork, test locally, and submit a PR from CONTRIBUTING.md
- [ ] Design decisions are captured in ADRs
- [ ] CLAUDE.md gives a Claude Code session enough context to work on the repo

## Ready to Tag: YES / NO
[If NO: list what blocks it]
```