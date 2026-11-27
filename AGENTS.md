# CostOps — agent instructions

Shared instructions for any AI coding agent (Claude Code, Codex, Copilot, …) working in this repository. Read `docs/agents/boundaries.md` before starting any task.

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues (ReinaldoLuizSilva/CostOps) via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.

### Agent boundaries

What agents may and must never do in this repository (no cloud writes, no real data, no merges, hands-on-first for Azure learning objectives). Read before starting any task. See `docs/agents/boundaries.md`.
