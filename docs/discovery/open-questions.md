# Open questions and risks

Tracked during product discovery. Resolved items move to `GLOSSARY.md` or `docs/adr/`.

## Risks

- **Unvalidated problem.** The multi-cloud cost-visibility pain is a hypothesis; it is not yet confirmed by anyone who administers multi-client environments.
- **Thin real data.** The only available environment is a personal Azure study subscription, with few resources and low spend. History comparison and anomaly detection cannot be meaningfully exercised on it alone.

## Mitigations agreed

- Unvalidated problem → parallel, non-code validation conversations (see `product-brief.md`).
- Thin real data → synthetic dataset as a first-class project artifact.

## Blockers to check

- **Azure offer type of the study subscription.** Azure for Students and Sponsorship are not supported by Cost Management APIs. Check in the portal: Subscriptions → Overview (Offer / Offer ID) and Cost Management + Billing → Properties (billing account type). See `research-cost-apis.md`.

## Open questions

- ~~Finality cutoff vs provider billing periods.~~ **Resolved (specification phase):** a provider-agnostic Re-collection Window of 45 days; see `product-brief.md`.
- **Cost basis when EffectiveCost is unavailable.** MOSP data has no amortized cost; under the no-fabrication rule EffectiveCost may be null. The specification picks one Cost Basis per Client per Report: EffectiveCost only when populated on every Cost Record in scope, otherwise BilledCost for everything, stated in the Report. Confirm after the spike.
- **ChargeCategory for MOSP.** The MOSP export has no charge type. It stays null unless the spike finds a provider-documented source or rule.
- **OIDC subject details.** Confirm on the first token exchange (M1) the exact immutable environment subject and whether Azure matches it case-sensitively.

## Pending decisions

- **Azure cost collection mechanism** — decided by a ~1 h spike on the study subscription before M2. Preference: **Exports to Blob** if the MOSP subscription offers the actual-cost dataset; fallback: **Query API**; rejected: legacy Consumption Usage Details API (deprecated except pay-as-you-go). Spike steps: (1) try creating an export in the portal and note available datasets/formats; (2) run a Query API call grouped by `ResourceId` at Daily granularity; (3) compare both totals with Cost Analysis. Write the ADR after the spike, with its results.
- **Second provider order (M8)**: leaning **OCI before GCP** — OCI FOCUS cost reports are GA and file-based (like Azure exports) and the Operator knows OCI; GCP's FOCUS export is Preview and requires BigQuery with per-query cost. Decide at M8.

## Backlog candidates captured during discovery

- **M1**: configure Claude Code deny rules (write-verb `az`, `terraform apply`/`destroy`, `git push` to `main`) — see `docs/agents/boundaries.md`.
- **Post-v0.1.0**: active notification via a private webhook (first item).
- **Post-v0.1.0**: flag cost decreases as anomalies.
- **Post-v0.1.0**: idle-resource heuristic "Resource with zero cost for N days".
- **Post-v0.1.0**: automated `terraform plan` on pull requests.
- **When needed**: presentation-time currency conversion with dated FX rates.
- **When needed**: run Collection Runs inside Azure (Functions / Container Apps Job) and private networking.
- **When needed**: database read model for a dashboard or API.
