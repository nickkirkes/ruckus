# Known Pitfalls

Project: Ruckus
Domain: Ruckus is a Claude Code plugin that implements gated development pipelines with subagent-per-task execution.

Pitfalls discovered through development. Updated by `/ruckus:build` and `/ruckus:fix` wrap-up stages.

---

## Domain-Specific

- **Agent files are plugin-shipped, not project-installed.** All 7 agents are loaded via `subagent_type` (e.g., `ruckus:code-reviewer`) from the plugin cache. The upgrade skill must never classify agent files as "New" or offer to copy them to `.claude/agents/`. The only valid agent-related check during upgrade is preamble drift on pre-existing `.claude/agents/` files.

## Data & State

- **CLAUDE.md as the source of truth for verify-all commands has two known failure modes.** Teams with mandated CLAUDE.md formats may not permit Roughly's Commands table, and third-party agents (claude-mem and others) that rewrite CLAUDE.md programmatically can clobber it. Session-length compaction is *not* a failure mode — skills explicitly `Read` CLAUDE.md at runtime per ADR-006, so the disk file is authoritative regardless of what was autoloaded. If breakage is reported, the clean fix is an additive `.ruckus/commands.md` fallback that skills check first, falling back to CLAUDE.md — strictly additive, no migration. Stay on CLAUDE.md until that happens to avoid a two-source-of-truth mental model.

## Integration

<!-- Pitfalls related to APIs, third-party services, cross-system communication -->

## Build & Deploy

<!-- Pitfalls related to build process, CI/CD, deployment -->

## Testing

<!-- Pitfalls related to test reliability, test data, flaky tests -->
