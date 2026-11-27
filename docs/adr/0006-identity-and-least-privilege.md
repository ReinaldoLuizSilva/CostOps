# CI uses a user-assigned managed identity federated to a GitHub Environment, with read-only cost access and container-scoped data writes

The scheduled workflow authenticates to Azure as a **user-assigned managed identity** with a federated credential whose subject is restricted to one GitHub Environment (e.g. `repo:<owner>/CostOps:environment:collect`), so forks and pull requests cannot obtain a token. The identity holds **Reader** on the subscription (cost data and Resource Graph inventory) and **Storage Blob Data Contributor** on the CostOps data container only (plus Storage Blob Data Reader on the exports container if exports are used). It has no permission to create resources or assign roles. Shared key access is disabled on the storage account, so only Entra ID authentication works and no connection strings or account keys exist. Infrastructure and role assignments are provisioned by the Operator locally with their own identity; local development also uses the Operator's own sign-in.

## Considered Options

- **App registration with a client secret**: rejected; a long-lived secret to store and rotate.
- **App registration with a federated credential**: viable, but managing app registrations needs Entra ID directory permissions outside subscription RBAC; a managed identity is an ordinary Azure resource governed by subscription RBAC and Terraform.
- **Cost Management Reader instead of Reader**: narrower, but it does not cover Resource Graph, which the Inventory Snapshot needs.
