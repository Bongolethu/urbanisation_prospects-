# Tasks: Urban/Rural Population Analytics — Fabric ETL

## Bronze
- [x] Create `notebooks/01_bronze_ingest_population.ipynb` with Fabric/local runtime bootstrap
- [x] Ingest `un_urban_rural_population_1950_2050.csv` -> `raw_urban_rural_population` (append, `batch_id`, `ingested_at`, `source_file`)
- [x] Ingest `countries_continents.csv` -> `raw_countries_continents`
- [x] Ingest `owid_country_to_who_regions.csv` -> `raw_who_regions`

## Silver
- [x] Create `notebooks/02_silver_clean_population.ipynb` with Fabric/local runtime bootstrap, attaching `bronze_lh` read-only
- [x] Dedupe each raw table on business key keeping max `batch_id`
- [x] Rename to snake_case, cast `year` to int, population columns to long
- [x] Compute `total_population`, `is_urban_estimated`, `is_rural_estimated`
- [x] Left-join latest `continent` and `who_region` per `entity`
- [x] Write `population_urbanization` (full overwrite)

## Gold
- [x] Create `notebooks/03_gold_urbanization_analytics.ipynb`, default lakehouse `gold_lh`, attaching `silver_lh` read-only (never `bronze_lh`)
- [x] Compute `urbanization_by_entity_year` (shares + YoY growth window function)
- [x] Compute `urbanization_by_continent_year` rollup
- [x] Compute `urbanization_by_who_region_year` rollup

## Orchestration & Environment
- [x] Add `pipelines/pl_urbanisation_etl.json` Fabric Data Pipeline (sequential Notebook activities)
- [x] Add `requirements.txt` entries for local execution (`pyspark`, `delta-spark`)

## Verification
- [x] Run all three notebooks locally end-to-end against `data/` and confirm row counts / no duplicate `(entity, year)` keys in Silver and Gold
- [x] Re-run the full pipeline a second time and confirm output tables are byte-for-byte identical (idempotency)
