# AI agent boundaries

Rules for any AI agent working in this repository. The Operator wants to understand and practise the Azure configuration, not only delegate it, and the repository is public.

## Agents may

- Write code, tests, documentation and Terraform, always on a branch and through a pull request.
- Generate and evolve the Synthetic Dataset.
- Run local tooling: tests, linters, `terraform fmt`, `terraform validate`, and `terraform plan` only when the Operator asks.
- Research documentation and perform reviews.

## Agents must never

- Run `terraform apply` / `terraform destroy`, or any `az` (or other cloud CLI) command that creates, changes or deletes resources, role assignments or permissions.
- Merge, push to `main`, or force-push.
- Read real data: Raw Extracts, Cost Records or Reports from a real subscription, or Terraform state. Contract fixtures reach an agent only after the Operator has anonymized them.
- Receive, print or handle secrets.

## Hands-on first, then code

For every Azure configuration that is a learning objective, the Operator performs it manually first (portal or CLI); only then does an agent help codify it in Terraform. This applies to: the collection spike, the storage account and its security settings, the managed identity with OIDC federation, role assignments, the budget, and the lifecycle policy.

## Technical enforcement

Written rules are backed by Claude Code permission deny rules for write-verb `az` commands, `terraform apply`/`destroy`, and `git push` to `main` (to be configured in M1). The Operator merges every pull request after reading the diff.
