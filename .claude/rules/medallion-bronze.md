---
paths:
  - "**/bronze/**/*.py"
  - "**/bronze/**/*.ipynb"
---
# Medallion Architecture: Bronze Stage (Raw Ingestion)

The Bronze layer is your raw data landing zone. Do not apply complex business transformations here.

## Core Directives
- **Append-Only:** Treat the Bronze layer as append-only. Keep a historical record of all ingested data, including a metadata timestamp of ingestion (`ingested_at`).
- **No Schema Enforcement:** Do not filter out malformed or partial records here. Capture the raw input exactly as it arrives from the source (APIs, databases, flat files).
- **Format:** Store Bronze data in robust, schema-preserving raw formats like Parquet or Delta Lake rather than mutable CSVs where possible.
- **Example Ingestion Pattern:**
  ```python
  # Always add metadata fields
  df = df.with_columns(
      pl.lit(datetime.now()).alias("ingested_at"),
      pl.lit("source_api").alias("data_source")
  )