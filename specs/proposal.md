# Proposal: Urban/Rural Population Analytics

## Stakeholders
- Analytics/Reporting consumers of the Gold layer (need urbanization trend insights per country/entity).
- Pipeline maintainers (own Bronze/Silver/Gold code and idempotency guarantees).

## Core Goals
1. Ingest raw urban/rural population CSV file(s) from a local `data/` folder into Bronze, unmodified, with ingestion metadata.
2. Clean, deduplicate, and standardize the data in Silver, producing a trustworthy single source of truth keyed by `Entity` + `Year`.
3. Expose a Gold analytics view over urbanization trends — e.g. urban population share, and urban vs. rural population growth, per `Entity` over time.

## Known Sources (`data/` folder)
- **`un_urban_rural_population_1950_2050.csv`** (UN World Urbanization Prospects 2018) — core fact table:
  - `Entity` (country/region name)
  - `Year`
  - `Urban population 1950-2050 (UN World Urbanization Prospects 2018)`
  - `Rural population 1950-2050 (UN World Urbanization Prospects 2018)`
- **`countries_continents.csv`** — dimension/lookup: `Entity`, `Year`, `Countries Continents`.
- **`owid_country_to_who_regions.csv`** — dimension/lookup: `Entity`, `Year`, `WHO region`.
- **`wid_pretax_income.csv`** — income inequality data (Gini, Palma ratio, percentile income shares). **Out of scope** — unrelated to urbanization; not part of this pipeline.
- No refund, transaction, or product-category concept applies to any of these — the earlier refund-analytics framing does not apply here and has been dropped.

## Open Question
- Should Gold roll up urbanization metrics by **continent** / **WHO region** using the two lookup files, in addition to per-`Entity` metrics? (Pending confirmation.)

## Known Constraints / Assumptions
- **Business key:** `Entity` + `Year` uniquely identifies a row; used as the Silver dedup key.
- **PII:** None expected — data is aggregate population counts per country/region, no individual-level records. No masking/hashing needed.
- **Data folder:** Bronze reads all CSV files placed under `data/` (not just the one file named above), so the pipeline should generalize to additional files with the same shape landing in that folder.
- **Types:** `Year` should be cast to integer; population columns cast to numeric (they may arrive as strings or contain blanks for years without data/estimates).
- **Nulls:** Some entities may lack urban or rural figures for certain years (e.g. projections vs. historical); Silver must flag or default these rather than silently dropping rows.
- **Gold metric definition:** Assuming "urbanization" is expressed as urban population as a share of total (urban + rural), plus raw growth trends — open to confirmation once Gold design is scoped.

## Out of Scope
- Real-time ingestion (static/batch CSV files only).
- Sub-national geographic breakdowns beyond what `Entity` provides.
