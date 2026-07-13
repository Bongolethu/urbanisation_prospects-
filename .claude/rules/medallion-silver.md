 
### File 5: `.claude/rules/medallion-silver.md`
*Applies only when Claude works within your cleaning and standardization layer.*

```markdown
---
paths:
  - "**/silver/**/*.py"
  - "**/silver/**/*.ipynb"
---
# Medallion Architecture: Silver Stage (Cleaned & Standardized)

The Silver layer represents validated, conformed, and enriched data. It serves as an enterprise-wide "single source of truth."

## Core Directives
- **Deduplication:** Always deduplicate raw Bronze records using a reliable business key.
- **Type Safety & Schema Validation:** Enforce strict data types. Convert raw strings to proper Datetime, Numeric, or Boolean types. 
- **Null Handling:** Convert unexpected nulls to sensible defaults or flag them. Do not let corrupt records slip past the Silver layer.
- **Anonymization:** Mask, hash, or strip out any PII (Personally Identifiable Information) during the Silver transformation.
- **Consistency:** Ensure standardized naming conventions (e.g., convert all column headers to `snake_case`).