# CI uses a user-assigned managed identity federated to a GitHub Environment, with read-only cost access and container-scoped data writes

The scheduled workflow authenticates to Azure as a **user-assigned managed identity** with a federated credential whose subject is restricted to one GitHub Environment (`repo:<owner>@<owner_id>/CostOps@<repo_id>:environment:collect`, see Amendments), so forks and pull requests cannot obtain a token. The identity holds **Reader** on the subscription (cost data and Resource Graph inventory) and **Storage Blob Data Contributor** on the CostOps data container only (plus Storage Blob Data Reader on the exports container if exports are used). It has no permission to create resources or assign roles. Shared key access is disabled on the storage account, so only Entra ID authentication works and no connection strings or account keys exist. Infrastructure and role assignments are provisioned by the Operator locally with their own identity; local development also uses the Operator's own sign-in.

## Considered Options

- **App registration with a client secret**: rejected; a long-lived secret to store and rotate.
- **App registration with a federated credential**: viable, but managing app registrations needs Entra ID directory permissions outside subscription RBAC; a managed identity is an ordinary Azure resource governed by subscription RBAC and Terraform.
- **Cost Management Reader instead of Reader**: narrower, but it does not cover Resource Graph, which the Inventory Snapshot needs.

## Amendments (2026-11-27, specification phase)

- **Immutable OIDC subject.** This repository issues GitHub OIDC tokens with immutable owner/repository IDs (repositories created after 2026-07-15). The federated credential subject is therefore `repo:<owner>@<owner_id>/CostOps@<repo_id>:environment:collect`, not the legacy `repo:<owner>/CostOps:environment:collect`. Azure accepts any subject at creation and only fails at token exchange, so the exact subject is confirmed on the first token exchange in M1. See `docs/discovery/research-cost-apis.md`.
- **One container per kind of data.** Azure Storage RBAC cannot scope by prefix without ABAC conditions, so data is split into containers and the CI identity's data-plane roles are granted per container:

| Container | Holds | CI identity | Operator |
|---|---|---|---|
| `config` | Client Registry | Storage Blob Data Reader | Storage Blob Data Contributor |
| `raw` | Raw Extracts (lifecycle-expired) | Storage Blob Data Contributor | Storage Blob Data Contributor |
| `cost-records` | Cost Record partitions | Storage Blob Data Contributor | Storage Blob Data Contributor |
| `inventory` | Inventory Snapshots | Storage Blob Data Contributor | Storage Blob Data Contributor |
| `reports` | Reports | Storage Blob Data Contributor | Storage Blob Data Contributor |
| `exports` (only if Exports are chosen) | Provider export files | Storage Blob Data Reader | Storage Blob Data Contributor |
