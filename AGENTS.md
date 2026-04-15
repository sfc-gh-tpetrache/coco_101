# Project Summary
This repo contains a Snowflake demo for analyzing support tickets. The demo classifies tickets into product categories, issue types, and priority buckets, then shows outputs useful for product managers and support managers. The main deliverable is a notebook that demonstrates Snowflake AI functions on support data.

# Stack and Layout
- Snowflake SQL
- Snowflake AI functions: AI_CLASSIFY, AI_COMPLETE
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

# Working Rules
- Treat the notebook as the main demo artifact
- Keep taxonomy stable unless the task explicitly changes it
- Prefer AI_CLASSIFY for fixed-label outputs and AI_COMPLETE for summaries or rationales
- Keep raw ticket text unchanged and place derived outputs in separate columns or result sets
- Keep the demo simple, inspectable, and safe for non-prod data only

# References
- See `docs/taxonomy.md`
- See `docs/prompts.md`
- See `docs/company-brief.md`
- See `docs/pipeline.md`