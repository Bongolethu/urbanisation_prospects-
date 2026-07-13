🛠️ Tutorial: Adding a New Data Source to the Pipeline
This tutorial walks through using Claude Code to implement a new data requirement into our Medallion framework from scratch using spec-driven development.

Step 1: Fork and Clone
Click Use this template on GitHub to create your own fork.

Clone your fork locally and navigate into the directory:

Bash
git clone <your-fork-url> && cd <your-fork-repo>
Initialize Claude Code inside your repo terminal:

Bash
claude
Step 2: Kick off Phase 1 (The Analyst Phase)
Ask Claude to add a new tracking requirement.

User Prompt:
"/analyst We need to add a new upstream marketing API dataset that tracks active user campaign clicks."

What Claude does: Claude checks .claude/rules/scaffolding-routing.md, adopts the Analyst persona, and creates specs/proposal.md. It will ask you clarifying questions about the data schema and business purpose rather than writing Python code.

Your Action: Answer its questions. Once satisfied, tell Claude: "The proposal looks perfect, let's lock it in."

Step 3: Shift to Phase 2 (The Architect Phase)
With the proposal aligned, move into system design.

User Prompt:
"/architect Design the schemas for this campaign dataset across our Bronze, Silver, and Gold layers."

What Claude does: Claude transitions to the Architect persona, opens .claude/rules/medallion-bronze.md, -silver.md, and -gold.md, and maps out a data architecture blueprint inside specs/design.md.

Design Output:

Bronze: Raw JSON landing with an added ingested_at timestamp.

Silver: Deduplicated campaign clicks keyed by campaign_id + user_id, cast strings to proper datetimes.

Gold: High-level summary fact tables (total_clicks_per_campaign).

Step 4: Map the Work and Execute (The Developer Phase)
Finally, break the blueprint down into physical execution tasks.

User Prompt:
"/developer Generate the execution tasks file and implement the Bronze ingestion stage."

What Claude does: It generates a checkbox checklist inside specs/tasks.md. It then reads .claude/rules/medallion-bronze.md and creates src/bronze/ingest_marketing.py.

Self-Validation: Thanks to your .claude/settings.json, Claude will automatically run ruff check . to clean up code formatting mistakes without pausing to ask you for permission.

Step 5: Verify Progress
You can check specs/tasks.md at any time to see exactly which pipeline components Claude has completed and tested!