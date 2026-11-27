# CostOps

CostOps builds a consolidated, historical view of cloud resources and their costs across providers, so that whoever administers those environments can see what exists, what it costs, and what changed.

## Language

### People and organizations

**Operator**:
The person who runs a CostOps installation and reads its output. In the MVP there is exactly one Operator.
_Avoid_: User, admin

**Client**:
An organization whose cloud environments are analyzed by CostOps; every collected fact belongs to exactly one Client. The MVP has a single Client, but the concept exists from the start.
_Avoid_: Customer, tenant, organization

**Client Registry**:
The explicit declaration of the Clients and the Sub Accounts that belong to each. Only declared Sub Accounts are collected; Sub Accounts are never auto-discovered.
_Avoid_: Config, account list, discovery

### Provider accounts

**Billing Account**:
The provider-side entity that receives the invoice (an Azure billing account, a GCP billing account, an OCI tenancy). One Billing Account may pay for Sub Accounts of several Clients.
_Avoid_: Payer, invoice account

**Sub Account**:
The provider-side container that holds Resources and is the unit of collection: an Azure subscription, a GCP project, an OCI tenancy. Each Sub Account belongs to exactly one Client.
_Avoid_: Subscription, project, account (when speaking generically)

### Inventory and cost

**Resource**:
A provisioned object in a cloud provider, identified by the provider's own resource identifier, normalized by that provider's comparison rule (for Azure, case-insensitive). The original form is kept for display.
_Avoid_: Asset, item, service instance

**Inventory Snapshot**:
The set of Resources that existed in one Sub Account at one point in time, with their descriptive attributes (name, type, region, grouping, tags). One is taken per Collection Run and kept as history.
_Avoid_: Inventory list, asset list

**Cost Record**:
A monetary charge for a period, attributed to a provider service and, when the provider allows it, to a Resource. Cost Records and Inventory Snapshots are independent: a charge may have no Resource, a Resource may have no charge, and a charge may refer to a Resource that no longer exists.
_Avoid_: Bill line, invoice item, usage record

**Billing Currency**:
The currency in which the provider invoiced a Cost Record. Amounts are always kept in their Billing Currency and never converted when stored.
_Avoid_: Currency (unqualified), local currency

### Time and data finality

**Charge Period**:
The span of time a Cost Record covers. In CostOps it is always one calendar day in UTC.
_Avoid_: Usage date, billing date

**Comparison Period**:
The unit used to compare cost over time: a calendar month in UTC.
_Avoid_: Billing period (that is the provider's invoice cycle, which may not align with calendar months)

**Provisional**:
The state of a Cost Record whose value the provider may still restate (open month or still within the provider's adjustment window). Provisional Cost Records are replaced on re-collection.
_Avoid_: Estimated, draft

**Final**:
The state of a Cost Record whose period has closed past the cutoff; it is no longer re-collected.
_Avoid_: Closed, settled

**Incomplete Day**:
One of the most recent Charge Periods whose cost is known to be only partially reported by the provider. Incomplete Days are shown as such and excluded from comparisons and anomaly detection.
_Avoid_: Partial day, pending day

### Collection

**Collection Run**:
One execution of CostOps that fetches provider data for a set of Sub Accounts and Charge Periods and turns it into Cost Records and Inventory Snapshots.
_Avoid_: Job, sync, import

**Raw Extract**:
The provider data received in one Collection Run, kept exactly as received, before any normalization.
_Avoid_: Dump, landing data, raw file

**Synthetic Dataset**:
Generated, deterministic Raw Extracts shaped exactly like a provider's real data, describing a fictitious Client and its scenarios (baseline, injected spike, new service, deleted Resource, restatement, Incomplete Days, second currency). It never contains real data.
_Avoid_: Mock data, fake data, sample data

### Analysis

**Baseline**:
The expected daily cost of one (Sub Account, service, Billing Currency) series, derived from its recent complete history.
_Avoid_: Average, normal cost, forecast

**Cost Anomaly**:
A complete day on which a series' cost deviates from its Baseline beyond both a statistical threshold and a minimum absolute amount, or on which a service shows cost for the first time in a Sub Account above that minimum. A Cost Anomaly only informs; it never triggers actions on Resources.
_Avoid_: Spike, alert, outlier

**Report**:
The periodic summary of one Client's cost: data freshness, month-to-date versus the same span of the previous month, top services and largest changes, Cost Anomalies, and Inventory Mismatches.
_Avoid_: Dashboard, statement, invoice

**Inventory Mismatch**:
A disagreement between Inventory Snapshot and Cost Records: a Resource with no cost, or cost attributed to a Resource absent from the latest Inventory Snapshot.
_Avoid_: Orphan, drift
