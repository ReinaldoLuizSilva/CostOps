# Product brief

Living summary of product decisions made during discovery. Architectural decisions live in `docs/adr/`; vocabulary lives in `GLOSSARY.md`.

## Shape

Internal, self-hosted tool run by one Operator over the cloud environments of one or more Clients. SaaS is out of scope (ADR-0001).

## Status of the problem

The multi-cloud cost-visibility problem is a **hypothesis**. It is validated in parallel, outside the code, through 2–3 conversations with people who administer cloud costs. Their outcome may redirect the roadmap from M8 onward.

## v0.1.0 success criterion

v0.1.0 is a **reference implementation and learning lab**, not a validated product. It is done when:

1. A scheduled job collects real data from the Operator's Azure study subscription.
2. History accumulates across runs without duplication.
3. A report is produced from the stored history.
4. An anomaly **injected into a synthetic dataset** is detected.

A realistic synthetic dataset is a first-class part of the project: it enables local testing without Azure, keeps real billing data out of the public repository, and provides enough history to exercise period comparison and anomaly detection.

## MVP scope (v0.1.0)

In scope:

- Cost Records as the primary data: consolidation, history, period-over-period comparison.
- A thin Inventory Snapshot, used to enrich Cost Records (tags, type, region) and to surface mismatches: Resources with no cost, and costs for Resources that no longer exist.
- A scheduled report.
- Cost anomaly detection.

Out of scope for v0.1.0:

- **Idle resource detection.** It needs usage metrics and per-resource-type rules. A cheap heuristic ("Resource with zero cost for N days") is a candidate first step after v0.1.0.
- **Cost allocation** (splitting a Sub Account's cost between Clients by tag or rule).

## Positioning against native tooling

Azure already provides anomaly alerts, budgets and scheduled cost emails, for Azure only. CostOps does not try to replace them; its value is **multi-cloud consolidation and a history owned by the Operator**.

## Time granularity and data finality

- Cost Records are daily (UTC Charge Periods); comparisons use calendar months.
- Each scheduled run re-collects every charge month that overlaps the Re-collection Window (default 45 days, configurable). A charge month becomes Final once its last day falls outside the window. This is provider-agnostic and covers billing cycles anchored on any day (decided in the specification phase, replacing the earlier "day N of the next month" cutoff).
- Final partitions are rewritten only by a Targeted Re-collection explicitly requested by the Operator (also used for initial backfill).
- The most recent ~3 days are Incomplete Days: flagged in reports and excluded from anomaly detection.

## Currency

- Amounts are stored in their Billing Currency and **never converted at storage time** (conversion would mix FX movement with cost movement and lose the invoiced value).
- Aggregations, comparisons and anomaly detection operate per currency; reports with mixed currencies show per-currency subtotals and no grand total.
- Presentation-time conversion with dated FX rates is a backlog item, triggered by a real need (e.g. GCP billing in USD).

## Architecture principles agreed so far

- Each provider's collector is the **only** code that knows that provider's data format; its output is always Cost Records in the FOCUS subset (ADR-0002). This isolates the choice of collection mechanism.

## Cost anomaly detection (v0.1.0)

- Unit of analysis: (Sub Account, service, Billing Currency), with the Sub Account total as an additional signal.
- Baseline: median of the last 28 complete days; anomaly when the **increase** over the Baseline exceeds both k × MAD and a configurable minimum absolute amount. Decreases are not flagged in v0.1.0 (backlog).
- Edge rules: MAD = 0 → only the minimum amount applies; only Complete days are analysed, and a Complete day with no Cost Records counts as zero; Incomplete Days and Collection Gaps are neither evaluated nor used in the Baseline; negative amounts (credits) count in the net daily value; fewer than 14 Complete days → "insufficient history" instead of a verdict.
- One Cost Basis per Client per Report: EffectiveCost only when populated on every Cost Record in scope, otherwise BilledCost for everything.
- "New cost" rule: a service with cost for the first time in a Sub Account, above the minimum amount.
- Uses `EffectiveCost`; Incomplete Days are excluded; no evaluation with fewer than 14 days of history.
- Anomalies are recorded and surfaced in the report/notification; no automatic action on Resources.
- k, minimum amount and windows are configurable and tuned against the synthetic dataset.
- Month-over-month comparison is a report feature, not anomaly detection.

## Report (v0.1.0)

- Per Client: data freshness (last complete day, Provisional state, last Collection Run), month-to-date total, and a month-to-date comparison on matched days (day numbers Complete in both months) with its coverage stated (per currency), top services and largest changes, Cost Anomalies, Inventory Mismatches.
- Format: Markdown plus CSV tables, written to a private `reports/` container.
- Never published to the repository, workflow artifacts, GitHub Pages or logs. The job summary shows only success/failure.
- **Accepted limitation:** no active notification in v0.1.0; the Operator opens the Report to see anomalies. Active notification via a private webhook is the first post-v0.1.0 item.

## Testing strategy

- The Synthetic Dataset generator emits provider-shaped Raw Extracts (Azure shape per the collection spike), deterministic by seed, with scenarios: weekly-seasonal baseline, injected spike, new service, deleted Resource still billed, restatement on re-collection, Incomplete Days, second currency. It drives the real pipeline end to end.
- Layers: unit (Azure→FOCUS mapping, anomaly math, finality rules); contract (real response shapes captured in the spike, **anonymized by hand** before committing); local integration (full pipeline on the Synthetic Dataset with local-disk storage); manual live smoke run (`workflow_dispatch`) against the study subscription.
- Every PR runs `ruff check`, `ruff format --check`, `pytest`, `terraform fmt -check`, `terraform validate`. No PR step touches Azure.
- Static type checking (mypy/pyright) is left to the spec.

## Security baseline (from the first commit)

- **Repository**: `.gitignore` blocks local data (`data/`, `reports/`, `*.parquet`), Terraform state/vars (`*.tfstate*`, `*.tfvars`, `.terraform/`) and `.env`; GitHub secret scanning and push protection on; a fixture-guard test fails if test files contain real-looking GUIDs outside the agreed fictitious values; subscription/tenant/client IDs live as GitHub Environment `collect` variables, never in code.
- **CI / supply chain**: `main` protected (PR + green CI); Environment `collect` restricted to `main`; actions pinned by commit SHA; Dependabot for pip and actions; minimal `permissions:` per workflow (`id-token: write` only where OIDC is used).
- **Azure storage**: see ADR-0008.
- **Accepted risks**: storage reachable from the internet behind identity only; public repository means public logs, mitigated by never printing data.

## Inventory Snapshot (v0.1.0)

- One snapshot per Collection Run (daily), stored as history per (Client, Sub Account, snapshot date); no retention limit in the MVP (negligible volume).
- Attributes: resource id, name, type, region, resource group (or provider grouping), tags. No detailed configuration (SKU, properties) until the idle heuristic needs it.
- Inventory Mismatches are computed against the latest snapshot.
- Resource identity uses the provider's normalized id (Azure: case-insensitive, compared in lowercase; original kept for display). The rule is documented in the Azure mapping.

## Documentation language

- All repository documentation, including the README, is in English. The README is rewritten at the close of M0 to reflect discovery decisions (vision, MVP scope, architecture with ADR links, security principles, revised roadmap, planning status). No parallel pt-BR README.

## Roadmap (revised 2026-11-27)

Capacity: ~4 h/week; effort estimate for the agreed v0.1.0 scope is ~38–49 h, which does not fit the original 14/02/2027 target without cutting learning or colliding with the AZ-104 exam (end of February 2027). The roadmap is reorganized into vertical slices, with an explicit exam pause. Dates are targets, not commitments.

| Milestone | Goal | Target |
|---|---|---|
| M0 | Planning & Architecture: discovery docs, commit, README rewrite | 2026-12-06 |
| M1 | Foundations & access: repo skeleton, CI, security baseline, agent deny rules, manual Azure setup (storage, managed identity, OIDC), collection spike + its ADR | 2026-12-20 |
| M2 | Local walking skeleton: real cost → Raw Extract → FOCUS → Parquet → minimal Report | 2027-01-10 |
| M3 | Synthetic Dataset + history, Provisional/Final, idempotency | 2027-01-24 |
| M4 | Automation: Terraform + scheduled OIDC workflow writing to Blob (success criteria 1–3 met) | 2027-02-07 |
| — | AZ-104 pause (0–2 h/week on CostOps) | end of Feb 2027 |
| M5 | Inventory Snapshot + Inventory Mismatch + full Report | 2027-03-14 |
| M6 | Cost Anomaly detection (success criterion 4 met) | 2027-03-28 |
| M7 | Release v0.1.0 | 2027-04-04 |
| M8+ | Second provider, informed by hypothesis validation | TBD |

**Checkpoint at M4 (2027-02-07):** if more than 2 weeks late, Inventory Snapshot and Inventory Mismatch move to v0.2.

## Client Registry

- Clients and their Sub Accounts are declared explicitly in a configuration file kept in the private Blob container (`config/costops.yaml`); locally, an equivalent file outside git. The repository holds only `costops.example.yaml` with the Synthetic Dataset's fictitious values.
- No auto-discovery: a Sub Account visible to the identity but not declared is skipped, and only the count of undeclared Sub Accounts is logged (never their ids). This preserves "every Sub Account belongs to exactly one Client" (ADR-0001).
