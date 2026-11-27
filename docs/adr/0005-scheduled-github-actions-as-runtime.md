# Collection Runs execute as a scheduled GitHub Actions workflow, authenticated to Azure via OIDC

In the MVP the scheduled Collection Run is a GitHub Actions workflow (`schedule: cron`) that authenticates to Azure with workload identity federation (OIDC), so no Azure secret is stored anywhere. It needs no compute to provision or pay for and matches the project's size. Running in Azure (Functions with a timer trigger, or a Container Apps Job) was considered and deferred: both add hosting, deployment and monitoring surface the MVP does not need. Revisit if there is a concrete reason, such as private networking, long runtimes, or keeping sensitive processing out of a public repository's infrastructure.

## Consequences

Because the repository is public:

- Workflow logs are public: they must never contain cost amounts, subscription IDs, resource names or other identifiers.
- Workflow artifacts can be downloaded by any signed-in GitHub user: reports and data are never uploaded as artifacts.
- GitHub disables scheduled workflows in public repositories after 60 days without repository activity; this must be monitored.
- The CI identity gets read-only access to cost and inventory, and write access only to the CostOps data container.
