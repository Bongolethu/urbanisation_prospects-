---
paths:
  - "**/gold/**/*.py"
  - "**/gold/**/*.ipynb"
---
# Medallion Architecture: Gold Stage (Business-Ready)

The Gold layer contains highly aggregated, modeled data optimized for consumption.

## Core Directives
- **Dimensional Modeling:** Organize data into clean facts and dimension tables (Star Schema) or logical business aggregates.
- **Business Logic Only:** All transformations here must represent explicit business definitions (e.g., `active_monthly_users`, `churn_rate`).
- **Performance Optimization:** Index, partition, or optimize Gold tables for rapid query performance. Use storage-efficient types.
- **Aggregations:** Ensure mathematical aggregations account for missing historical records (e.g., using proper outer joins and replacing resultant nulls with `0`).