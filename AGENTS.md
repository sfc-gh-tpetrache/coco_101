# Project Summary
This repo contains a Snowflake demo for analyzing support tickets. The demo classifies tickets into product categories, issue types, and priority buckets, then shows outputs useful for product managers and support managers. The main deliverable is a notebook that demonstrates Snowflake AI functions on support data.

# Stack and Layout
- Snowflake SQL
- Snowflake AI functions: AI_CLASSIFY, AI_COMPLETE, AI_AGG
- Jupyter notebook for the end-to-end demo

Folders:
- `/data` — source CSV files (e.g. `support_data.csv`)
- `/notebooks` — main demo notebook
- `/sql` — setup scripts and optional dynamic-table examples
- `/docs` — taxonomy, prompts, and demo notes

# Verify Changes
- Run setup SQL in a dev or demo Snowflake environment
- Run the notebook end to end
- Validate label coverage, sample classifications, and summary quality
- Keep outputs aligned with the taxonomy in `docs/taxonomy.md`

Preferred commands:
- `snowsql -f sql/01_setup.sql`
- open and run `notebooks/support_ticket_demo.ipynb`

# Path Rules
- Always use relative paths (from the project root) in generated code, SQL scripts, and documentation — never absolute paths
- Exception: Snowflake `PUT` commands require absolute `file:///` paths (Snowflake limitation)

# Snowflake Python Guidelines
- Use Snowpark Session (`snowflake.snowpark.Session`) instead of `snowflake.connector` + cursor
- Use `session.sql(...).collect()` for DDL/DML and `session.sql(...).to_pandas()` for queries that feed charts or display
- Avoid pulling full datasets into pandas — let Snowpark push compute to Snowflake and only collect aggregated results at the visualization boundary
- For AI_COMPLETE, prefer performant frontier models (e.g. `claude-4-sonnet`, `claude-4-opus`) over smaller models like `mistral-large2`
- Check the latest supported model list at https://docs.snowflake.com/en/sql-reference/functions/complete-snowflake-cortex before choosing a model

# Working Rules
- Treat the notebook as the main demo artifact
- Keep taxonomy stable unless the task explicitly changes it
- Prefer AI_CLASSIFY for fixed-label outputs and AI_COMPLETE for per-row summaries or rationales
- Use AI_AGG for aggregating insights across many rows — it handles datasets larger than a single LLM context window and is the recommended function for batch, context-agnostic aggregation (e.g. "top 3 insights across all tickets"). Its instruction argument must be a string constant, not a column reference.
- Keep raw ticket text unchanged and place derived outputs in separate columns or result sets
- Keep the demo simple, inspectable, and safe for non-prod data only

# References
- See `docs/taxonomy.md`
- See `docs/prompts.md`
- See `docs/company-brief.md`
- See `docs/pipeline.md`