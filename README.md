# Ruckus

Gated development pipelines with subagent-per-task execution. Ruckus is a Claude Code plugin by Rowdy Cloud that makes agents prove their work at every stage — no quiet failures, no skipped steps, no hoping things work. Discover, plan, verify, implement, review, ship.

## Installation

```
/plugin install ruckus@rowdy-cloud
```

## Quick Start

```
/ruckus:setup
```

Setup detects your project's maturity level, collects essential context (stack, build commands, conventions), and creates the documentation structure that powers all Ruckus skills.

## Workflows

| Task | Command | Example |
|------|---------|---------|
| New feature | `/ruckus:build` | `/ruckus:build docs/epics/E02-story-3.md` |
| Bug fix | `/ruckus:fix` | `/ruckus:fix docs/issues/uat-issues.md ISSUE-001` |
| Code review | `/ruckus:review` | `/ruckus:review "Added auth flow"` |
| Epic review | `/ruckus:review-epic` | `/ruckus:review-epic docs/epics/E02.md` |
| Epic audit | `/ruckus:audit-epic` | `/ruckus:audit-epic docs/epics/E02.md` |
| Verify build | `/ruckus:verify-all` | `/ruckus:verify-all` |
| Plan review | `/ruckus:review-plan` | (dispatched by build/fix, also standalone) |
| Setup | `/ruckus:setup` | `/ruckus:setup` |
| Upgrade | `/ruckus:upgrade` | `/ruckus:upgrade` |

## Pipeline: `/ruckus:build`

The build pipeline drives feature implementation through 8 gated stages:

```
Intake → Discover → Plan → Review Plan → Implement → Review → Verify → Wrap-up
```

**Key behaviors:**
- Each task in the plan gets a fresh subagent (prevents context overflow)
- Two-stage review after every task (spec compliance + quality check)
- Plan review is mandatory and dispatched as a blocking subagent
- UI work detected per-task via `UI: yes/no` flag (loads frontend-design automatically)
- Human gates at every stage transition

## Pipeline: `/ruckus:fix`

Same 8-stage structure with investigation instead of discovery:

```
Intake → Investigate → Plan → Review Plan → Implement → Review → Verify → Wrap-up
```

**Key differences from build:**
- Stage 2 dispatches the investigator agent (or performs inline if agent doesn't exist)
- Compacts context before investigation to preserve headroom
- Commit messages use `fix:` prefix with issue ID reference
- Self-upgrades: offers to create investigator agent when project reaches 50+ files

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `discovery` | Sonnet | Research and scope features before planning |
| `investigator` | Sonnet | Diagnose bugs by tracing code paths |
| `epic-reviewer` | Opus | Cross-story epic review before implementation |
| `code-reviewer` | Sonnet | Bugs, security, conventions, anti-patterns |
| `static-analysis` | Sonnet | Type check, lint, build, dead code |
| `silent-failure-hunter` | Sonnet | Swallowed errors, missing handling, data integrity |
| `doc-writer` | Sonnet | Updates CLAUDE.md, known-pitfalls.md, ADRs |

## Maturity Levels

Ruckus adapts to your project's size:

| Level | Source Files | Behavior |
|-------|-------------|----------|
| Greenfield | <10 | Simplified verify-all, no investigator, no Stop hook |
| Scaffolded | 10-50 | Standard configuration |
| Established | 50+ | Full setup, investigator agent, Stop hook offered |

## Self-Upgrading

Ruckus checks for upgrade opportunities at the end of every build/fix run:

| Check ID | Trigger | Offers |
|----------|---------|--------|
| `investigator-v1` | 50+ source files, no investigator agent | Create investigator agent |
| `test-verify-v1` | Test config exists, verify-all test step is placeholder | Add test execution |
| `stop-hook-v1` | verify-all has 2+ meaningful checks, no Stop hook | Add Stop hook |
| `pitfalls-organized-v1` | known-pitfalls.md > 80 lines | Deduplicate and organize |

Check IDs are versioned. When a plugin update improves a check, the version bumps and previously-declined checks are re-offered with an explanation of what changed.

**Responses:** `yes` (apply) / `not yet` (ask again next run) / `never` (don't ask again for this version)

## Philosophy

**Loud.** Every gate produces visible output. No silent passes. If something fails, it makes noise until it's fixed.

**Gated.** No stage starts until the previous one proves itself. Plan review can't be skipped because it's a blocking subagent. Implementation can't start without verified plan approval.

**Opinionated.** Ruckus has opinions about how code should be built: discover first, plan in discrete tasks, verify the plan, implement one task at a time, review everything, verify the build. You can abort at any gate, but you can't skip one.

## Project Structure

```
ruckus/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── build/          (feature pipeline + subagent templates)
│   ├── fix/            (bug fix pipeline)
│   ├── review/         (parallel 3-agent review)
│   ├── review-epic/    (pre-implementation epic review)
│   ├── audit-epic/     (post-implementation epic audit)
│   ├── verify-all/     (type check + test + build loop)
│   ├── review-plan/    (plan verification)
│   ├── upgrade/        (update installed files)
│   └── setup/          (bootstrap + templates)
├── agents/             (7 agent definitions)
├── README.md
├── CHANGELOG.md
├── LICENSE
└── marketplace.json
```

## Contributing

1. Fork the repo
2. Create a feature branch
3. Make your changes (follow the gated pipeline — use `/ruckus:build` on itself)
4. Ensure all skills are under 300 lines and all agents under 500 words
5. Submit a PR

## License

MIT - Rowdy Cloud
