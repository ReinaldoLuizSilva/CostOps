# Terraform manages all CostOps Azure resources; state is remote in a dedicated storage account; apply is manual

Terraform manages every Azure resource that exists because of CostOps: resource group, data storage account (shared key disabled, data container, lifecycle policy for Raw Extracts), the CI managed identity and its federated credential, role assignments, the cost export if chosen, and a **subscription budget** with e-mail alerts (e.g. 50/80/100 % of R$ 100) that turns the cost constraint into an automatic guardrail. State is kept remotely in a dedicated storage account using Entra ID authentication (`use_azuread_auth`) and native locking; that account is created by a documented one-off bootstrap. State never enters the repository because it contains identifiers.

In the MVP, CI runs only `terraform fmt -check` and `terraform validate` (no Azure credentials). `plan`/`apply` are run manually by the Operator, because automating them would require a second CI identity with broad write and role-assignment permissions, contradicting ADR-0006. Automated `plan` on pull requests is a post-MVP evolution.

## Consequences

- The only permanent infrastructure is the data storage, the state storage, the identity and the budget: no always-on compute.
