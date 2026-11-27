# Research: cost APIs, FOCUS and Azure constraints

Verified against official documentation on 2026-11-27. Re-check before relying on any limit or price.

## FinOps FOCUS

- Latest released spec: **v1.4** (v1.5 announced). https://focus.finops.org/focus-specification/
- **Azure**: Exports offer a "Cost and usage details (FOCUS)" dataset (CSV/Parquet, daily MTD or monthly), Microsoft dataset "FOCUS 1.2-preview". **Only EA, MCA and MPA; MOSP (pay-as-you-go/free account created on the website) is not supported**, nor management-group scope. https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-improved-exports
- **GCP**: native FOCUS export to BigQuery, **Preview**; requires Billing Export to BigQuery; queries are billed. https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-focus-setup
- **OCI**: FOCUS CSV cost reports since 2024-06, every 6 h, retained 1 year, read from an Oracle-owned bucket via cross-tenancy policy; FOCUS version not stated. https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costusagereportsoverview.htm
- Key columns: BillingAccountId, SubAccountId (Azure subscription / GCP project), ResourceId, ServiceName, ServiceCategory, ChargeCategory, BilledCost vs EffectiveCost (amortized), BillingCurrency, ChargePeriodStart/End (end exclusive). https://learn.microsoft.com/en-us/cloud-computing/finops/focus/what-is-focus

## Azure Cost Management

- **Supported offers**: Pay-as-you-go, Free Trial, Dev/Test, MSDN/Visual Studio, Azure Pass, EA, MCA. **Not supported: Azure for Students, Students Starter, Sponsorship, classic CSP.** Credits are not shown. https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/understand-cost-mgt-data
- **Cost Details API**: EA/MCA only; for other account types use Exports (or the legacy Consumption Usage Details API). https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/manage-automation
- **Query API**: Daily or Monthly granularity, up to 2 group-by dimensions; per-tenant quota 12 QPU/10 s, 60/min, 600/h (1 QPU ≈ 1 month queried). Data refreshes every ~4 h; Microsoft recommends calling at most once per day.
- **Latency**: pay-as-you-go up to **72 h**; month closes ~72 h after period end and can change until day 5; open-month figures are estimates. Retention 13 months in portal/exports. New subscriptions need up to 48 h before features work.
- **Cost**: Cost Management features are free; exports only incur destination storage.

## Built-in overlap (Azure-only)

- **Anomaly detection**: subscription scope only, daily, ~36 h after day end, needs 60 days of history; alerts by email.
- **Budgets** and **scheduled emails** on saved views exist.
- None of these ingest other clouds.

## Resource Graph and RBAC

- Resource Graph is free; per-user throttling (~15 queries / 5 s); 1,000 rows per page.
- **Reader** at subscription scope covers both Resource Graph and cost reads. Creating exports/budgets/alerts needs Cost Management Contributor; exports also need write on the storage account.

## CI identity and storage

- GitHub Actions → Azure via workload identity federation (app registration or user-assigned managed identity): no secrets, free.
- Blob LRS hot ≈ US$0.02–0.03/GB-month (eastus / brazilsouth, Retail Prices API); a few GB with daily writes is well under US$1/month.

## Collection mechanism for MOSP pay-as-you-go (verified 2026-11-27)

- Microsoft recommends **Exports** for MOSP; the Cost Details API is not supported for it. https://learn.microsoft.com/en-us/azure/cost-management-billing/automate/get-usage-details-legacy-customer
- **Docs conflict**: the exports tutorial lists Actual/Amortized datasets only for EA/MCA/MPA; the only explicit MOSP statement is that FOCUS is unsupported. Must be confirmed manually in the portal. https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-improved-exports
- Export behaviour: daily month-to-date runs rewrite the month; a hidden run during days 1–5 refreshes the prior month; partitioned output with `manifest.json`; layout `Container/Dir/ExportName/[YYYYMMDD-YYYYMMDD]/[RunID]/`; "overwrite data" on by default.
- **MOSP export schema** (only version 2019-11-01) differs from EA/MCA: `UsageDateTime`, `InstanceId` (resource id), `PreTaxCost`, `Currency`, `MeterCategory`, `ServiceName`, `ResourceGroup`, `ResourceType`, `Tags` (no resource-group tags), no `ChargeType`. https://learn.microsoft.com/en-us/azure/cost-management-billing/dataset-schema/cost-usage-details-pay-as-you-go
- MOSP cost data excludes Marketplace and commitment purchases.
- **Query API**: up to 2 group-bys (Dimension or TagKey), Daily/Monthly, `nextLink` pagination; grouping by `ResourceId` at Daily granularity is only evidenced in Q&A, not reference docs; page size undocumented (~5,000 rows reported). https://learn.microsoft.com/en-us/rest/api/cost-management/query/usage
- **Consumption Usage Details API**: deprecated except for pay-as-you-go and Visual Studio; no retirement date.
- **Terraform**: `azurerm_subscription_cost_management_export` (CSV only, no identity/partition options). Creating exports needs Cost Management Contributor plus storage permissions; firewall-protected storage works via the export's system-assigned managed identity, but whether a Terraform-created export gets it is unverified.
- **Currency**: MS-AZR-0003P in Brazil bills in **BRL**. https://azure.microsoft.com/en-us/pricing/offers/ms-azr-0003p
- FinOps toolkit hubs (0.2+) do not support MOSP.

### To confirm manually (spike)

1. Which export datasets/formats the portal offers on the study subscription.
2. Query API page size.
3. Daily grouping by `ResourceId` on MOSP.
