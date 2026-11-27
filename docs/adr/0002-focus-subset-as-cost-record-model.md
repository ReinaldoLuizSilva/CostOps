# Cost Records follow a subset of FinOps FOCUS, without full conformance

The normalized Cost Record uses the column names and semantics of the FinOps Open Cost and Usage Specification (FOCUS), restricted to the columns the MVP needs: `BillingAccountId`, `SubAccountId`, `ResourceId`, `ServiceName`, `ServiceCategory`, `ChargeCategory`, `BilledCost`, `EffectiveCost`, `BillingCurrency`, `ChargePeriodStart`, `ChargePeriodEnd` (exclusive) and tags, plus CostOps-specific columns (`ClientId`, `Provider`, a collection identifier). FOCUS already encodes hard-won distinctions (billed vs amortized cost, charge categories, charge vs billing period, currency), GCP and OCI export it natively, and its vocabulary is recognized by FinOps practitioners; inventing our own model would rediscover those problems and make every new provider a bespoke mapping.

## Considered Options

- **Own model** (README draft: provider, account, service, resource, tags, cost, period): rejected, see above.
- **Full FOCUS conformance**: rejected; dozens of columns and validation rules are disproportionate for a ~40 h MVP.

## Consequences

- Azure's native FOCUS export is unavailable for MOSP (pay-as-you-go) billing accounts, so the Azure collector maps Cost Management API output to FOCUS itself. That mapping is documented per provider.
- `BilledCost` and `EffectiveCost` are both stored even when equal (no reservations/savings plans in the study subscription).
- History is persisted in this shape, so changing the model later means migrating stored data.
