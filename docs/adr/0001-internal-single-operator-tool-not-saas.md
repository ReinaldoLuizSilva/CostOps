# CostOps is an internal, self-hosted tool, not a SaaS

CostOps is designed as an internal, self-hosted tool run by one Operator over the cloud environments of one or more Clients (the consultancy/MSP shape). The MVP runs with a single Operator and a single Client, but Client is part of the domain model from day one so that adding more Clients later does not require reshaping the data. SaaS (external sign-up, self-service onboarding, billing, multi-tenant isolation) is explicitly out of scope: it does not fit a ~4 h/week solo budget and adds little Cloud/DevOps learning relative to its cost.

## Consequences

- No user accounts, sign-up, or per-tenant isolation layer will be built.
- Every collected record must be attributable to a Client, even while only one exists.
- The account hierarchy is Client → Billing Account → Sub Account (FOCUS terms). The Sub Account is the unit of collection and belongs to exactly one Client; a Client is attributed per Sub Account, never per Billing Account, so one Billing Account can serve several Clients.
- Provider-specific groupings (Azure resource group, OCI compartment, GCP folder) are attributes of Resources and Cost Records, not levels of the hierarchy. The Entra ID tenant is an authentication detail, not a domain concept.
- Splitting one Sub Account's cost between several Clients (cost allocation, showback/chargeback) is out of scope.
