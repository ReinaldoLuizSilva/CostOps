# History is stored as partitioned Parquet files in Blob Storage and queried with DuckDB; no database server

Cost Records are stored as one Parquet file per partition (Client, Sub Account, charge month), e.g. `cost-records/client=…/sub-account=…/month=YYYY-MM.parquet`, in an Azure Blob Storage container; replacing a partition is a single blob overwrite, which is atomic (ADR-0003). Raw Extracts are kept in their provider's original format under a separate prefix and expired by a lifecycle management policy. Reports and analysis query the files in-process with DuckDB (SQL, no server). Locally, the same layout lives on disk, so development and tests need no Azure access. Data volume is small (hundreds to a few thousand rows per Sub Account per month), cost is cents per month, and a database server would be infrastructure without a proven consumer.

## Considered Options

- **CSV/JSON files**: rejected for normalized data; they do not preserve types (decimal amounts, UTC timestamps), so every read must re-parse and re-validate.
- **SQLite file in Blob**: rejected; the whole database is rewritten each run and there is no natural separation between raw and normalized data.
- **Managed database** (Oracle ADB Always Free, Azure SQL, PostgreSQL Flexible): deferred. ADB is free and familiar but adds cross-cloud networking and wallet handling with little Azure learning; the others add cost or limits. Revisit when a real consumer (dashboard, API, multiple readers) exists.

## Consequences

- Parquet and DuckDB are added to the stack; both are justified by typed storage and SQL over files, not by the roadmap.
- Any future database is a read model loaded from these files, not a replacement of the history format.
