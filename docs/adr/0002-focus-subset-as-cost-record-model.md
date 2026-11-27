# Cost Records follow a subset of FinOps FOCUS, without full conformance

The normalized Cost Record uses the column names and semantics of the FinOps Open Cost and Usage Specification (FOCUS), restricted to the columns the MVP needs: `BillingAccountId`, `SubAccountId`, `ResourceId`, `ServiceName`, `ServiceCategory`, `ChargeCategory`, `BilledCost`, `EffectiveCost`, `BillingCurrency`, `ChargePeriodStart`, `ChargePeriodEnd` (exclusive) and tags, plus CostOps-specific columns (`ClientId`, `Provider`, a collection identifier). FOCUS already encodes hard-won distinctions (billed vs amortized cost, charge categories, charge vs billing period, currency), GCP and OCI export it natively, and its vocabulary is recognized by FinOps practitioners; inventing our own model would rediscover those problems and make every new provider a bespoke mapping.

## Considered Options

- **Own model** (README draft: provider, account, service, resource, tags, cost, period): rejected, see above.
- **Full FOCUS conformance**: rejected; dozens of columns and validation rules are disproportionate for a ~40 h MVP.

## Consequences

- Azure's native FOCUS export is unavailable for MOSP (pay-as-you-go) billing accounts, so the Azure collector maps Cost Management API output to FOCUS itself. That mapping is documented per provider.
- `BilledCost` and `EffectiveCost` are both stored even when equal (no reservations/savings plans in the study subscription).
- History is persisted in this shape, so changing the model later means migrating stored data.

## Amendments (2026-11-27, specification phase)

- **Reference version: FOCUS 1.2.** It is the version Microsoft's FOCUS dataset and GCP's FOCUS export document against; every column in the subset exists since FOCUS 1.0, so later versions remain reachable.
- **No fabricated values.** Each column is classified as Required, Optional, Declared (supplied by the Operator in the Client Registry), Derived (only by a rule documented by the provider or by this specification) or Unavailable (stored as null). A value the provider does not supply is never invented.
- **Internal schema version.** Every partition records the CostOps Cost Record schema version and the FOCUS reference version. A breaking schema change bumps the version and is migrated by re-normalizing from Raw Extracts (within retention) or by an explicit transformation.
