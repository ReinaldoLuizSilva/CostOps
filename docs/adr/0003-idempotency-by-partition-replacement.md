# Idempotency by atomic partition replacement, with immutable Raw Extracts

Every Collection Run produces the **complete** set of Cost Records for each partition it covers, where a partition is (Client, Sub Account, charge month), and replaces the previous content of that partition atomically. Partitions whose records are Final are never rewritten. The provider data received in each run is kept unchanged as a Raw Extract for a limited retention period (order of 90 days, to be fixed later). We chose this over row-level upserts because Azure's pay-as-you-go cost data has no unique line identifier (several legitimate rows can share resource, day and meter), so a natural-key upsert would either merge real rows or leave stale ones; partition replacement is idempotent by construction and mirrors how Azure itself rewrites the whole month on each export run.

## Considered Options

- **Upsert by natural key**: rejected, no reliable key per line.
- **Append-only with a version per run**: rejected; every query must pick the latest run per partition and volume grows without bound. Raw Extracts already answer "what did we know on day X" more cheaply.

## Consequences

- A re-run on the same day, or a retry after failure, yields the same stored state.
- Normalization can be re-executed from Raw Extracts without calling the provider again, as long as they are within retention.
- The storage technology must support replacing one partition atomically.
