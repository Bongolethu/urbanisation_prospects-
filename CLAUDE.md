# Medallion ETL & Scaffolding Template

This repository implements a standardized Python Medallion Architecture combined with a strict, phase-based multi-agent engineering workflow.

## Core Commands
- Run Full Pipeline: `python src/main.py`
- Run Ingestion (Bronze): `python src/bronze/ingest.py`
- Run Cleansing (Silver): `python src/silver/transform.py`
- Run Aggregation (Gold): `python src/gold/aggregate.py`
- Environment Setup: `poetry install` or `pip install -r requirements.txt`
- Run Code Checks: `ruff check .`
- Execute Validation Tests: `pytest`

## Scaffolding & Multi-Agent Workflows
- **Spec-Driven Development:** We use a structured, phase-based SDLC workflow. High-level feature sets or pipeline modifications must start with a proposal (`specs/proposal.md`), transition to a system design (`specs/design.md`), and break down into an execution plan (`specs/tasks.md`) before any Python ETL changes occur.
- **Scope Alignment:** If a request is complex or touches multiple medallion boundaries, prompt the user to initialize the phase specs instead of jumping directly into writing script files.

## Architectural Expectations
- **Data Isolation:** Never let a Gold transformation layer query Bronze data directly. All data must flow sequentially: **Bronze -> Silver -> Gold**.
- **Idempotency:** Pipelines must be completely idempotent. Re-running a stage with identical inputs must yield identical results without duplicating rows.