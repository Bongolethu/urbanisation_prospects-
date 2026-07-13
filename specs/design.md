# Design: Urban/Rural Population Analytics — Microsoft Fabric ETL

## Architectural Decisions
- **Deviation from root `CLAUDE.md` Core Commands:** those commands (`python src/bronze/ingest.py`, etc.) assume plain `.py` scripts. This build targets **Microsoft Fabric** and ships as **Jupyter notebooks** (Fabric's native ETL artifact) instead. Notebooks are structured so the same logic also runs locally (outside Fabric) for testing — see "Local execution" below.
- **Open proposal question resolved:** Gold **will** roll up by continent and WHO region (the lookup files are already in `data/` and the join is low-cost). This can be dropped later without touching Bronze/Silver if it turns out unwanted.
- **PII masking step from `.claude/rules/medallion-silver.md` does not apply**: this dataset is country-level aggregate population counts, no individual-level records. Documented as an explicit exception rather than silently skipped.
- **Bronze/Silver/Gold isolation** (`CLAUDE.md` mandate) is enforced physically, not just logically: each layer is a separate Fabric **Lakehouse**, and the Gold notebook never attaches the Bronze Lakehouse — it cannot query Bronze even by mistake.

## Fabric Workspace Artifacts

| Artifact | Name | Purpose |
|---|---|---|
| Lakehouse | `bronze_lh` | Raw landing zone (append-only) |
| Lakehouse | `silver_lh` | Cleaned, conformed, deduplicated |
| Lakehouse | `gold_lh` | Business-ready aggregates |
| Notebook | `01_bronze_ingest_population.ipynb` | Default lakehouse: `bronze_lh` |
| Notebook | `02_silver_clean_population.ipynb` | Default lakehouse: `silver_lh`; attaches `bronze_lh` (read-only) |
| Notebook | `03_gold_urbanization_analytics.ipynb` | Default lakehouse: `gold_lh`; attaches `silver_lh` (read-only). Never attaches `bronze_lh`. |
| Data Pipeline | `pl_urbanisation_etl` | Orchestrates the 3 notebooks sequentially (Notebook activities, `Succeeded` dependency chain) |

Source CSVs land in `bronze_lh`'s `Files/raw/` area (Fabric) — for local dev they are read directly from this repo's `data/` folder.

## Data Flow & Schemas

### Bronze (`bronze_lh`, Delta tables under `Tables/`, append-only)
Raw columns preserved as-is (as strings) plus ingestion metadata. No filtering, no type coercion.

- `raw_urban_rural_population`: `Entity`, `Year`, `Urban population 1950-2050 (UN World Urbanization Prospects 2018)`, `Rural population 1950-2050 (UN World Urbanization Prospects 2018)`, `source_file`, `batch_id`, `ingested_at`
- `raw_countries_continents`: `Entity`, `Year`, `Countries Continents`, `source_file`, `batch_id`, `ingested_at`
- `raw_who_regions`: `Entity`, `Year`, `WHO region`, `source_file`, `batch_id`, `ingested_at`

`wid_pretax_income.csv` is **not ingested** (out of scope per proposal).

Each run appends a new `batch_id`; Bronze is an audit log and is allowed to accumulate multiple snapshots of the same `(Entity, Year)` over time. Idempotency of the *pipeline's output* is enforced downstream in Silver, not by preventing Bronze appends.

### Silver (`silver_lh`, table `population_urbanization`, full overwrite each run)
snake_case, deduplicated on the latest `batch_id` per business key, typed, conformed:

| column | type | notes |
|---|---|---|
| `entity` | string | business key part 1 |
| `year` | int | business key part 2 |
| `urban_population` | long, nullable | null if source blank |
| `rural_population` | long, nullable | null if source blank |
| `total_population` | long, nullable | `urban_population + rural_population`; null only if both are null |
| `is_urban_estimated` | boolean | true if `urban_population` was null/blank in source |
| `is_rural_estimated` | boolean | true if `rural_population` was null/blank in source |
| `continent` | string, nullable | left-joined from latest `raw_countries_continents` snapshot per `entity` |
| `who_region` | string, nullable | left-joined from latest `raw_who_regions` snapshot per `entity` |
| `_silver_loaded_at` | timestamp | processing metadata |

Dedup rule: within each business key `(entity, year)`, keep the row from the **max `batch_id`** (last write wins). This makes re-running Bronze+Silver with unchanged input fully idempotent — Silver always recomputes to the same state.

### Gold (`gold_lh`, full overwrite each run, reads Silver only)
- `urbanization_by_entity_year`: `entity`, `year`, `urban_population`, `rural_population`, `total_population`, `urban_share_pct`, `rural_share_pct`, `urban_population_yoy_growth_pct`
- `urbanization_by_continent_year`: `continent`, `year`, `total_urban_population`, `total_rural_population`, `avg_urban_share_pct`
- `urbanization_by_who_region_year`: `who_region`, `year`, `total_urban_population`, `total_rural_population`, `avg_urban_share_pct`

`urban_share_pct` = `urban_population / total_population * 100`, null-safe (null if `total_population` is null or 0). YoY growth computed via a window function partitioned by `entity`, ordered by `year`.

## Local Execution (outside Fabric)
Each notebook opens with a bootstrap cell that detects the runtime:
```python
try:
    import notebookutils  # only importable inside Fabric
    IN_FABRIC = True
except ImportError:
    IN_FABRIC = False
```
When `IN_FABRIC` is `False`, a local `SparkSession` (with `delta-spark`) is created, and Lakehouse `Tables/` paths are mapped to `./lakehouse/<bronze|silver|gold>/Tables/<table>` on local disk; raw source reads point at this repo's `data/` folder instead of `Files/raw/`. This lets the same notebook logic run under `pytest`/manually without a real Fabric workspace.

## Idempotency Check
- Bronze: re-running produces a new `batch_id` but does not corrupt downstream state (Silver always takes latest batch per key).
- Silver: full deterministic overwrite from Bronze — re-running with unchanged Bronze yields an identical table.
- Gold: full deterministic overwrite from Silver — same guarantee.
