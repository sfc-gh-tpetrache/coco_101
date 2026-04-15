# SnowCare — Support Ticket Intelligence

Classifies, enriches, and aggregates support tickets using Snowflake AI functions (`AI_CLASSIFY`, `AI_COMPLETE`, `AI_AGG`) and Dynamic Tables — turning 16,000 tickets/year into Monday-morning insights for two personas.

---

## The Story

**SnowCare** is a B2B SaaS analytics platform with ~4,200 customers and a support team of 12 agents. Today they spend **15 hours/week** manually reading, tagging, and routing tickets. The quarterly reporting cycle means Product gets insights **three months too late**. Feature requests, competitor mentions, and cross-module issues are buried in free text and never surfaced.

Two stakeholders need help:

- **Sarah Chen, VP of Customer Support** — *"I need to open something Monday morning and instantly know: what's breaking, who's upset, and where my team should focus this week."*
- **David Park, Director of Product** — *"Our customers tell support things they'd never tell us in a product survey. Feature requests, competitor comparisons, workaround hacks — it's all in the ticket data. We're just not mining it."*

This demo replaces manual triage with three Snowflake AI functions running inside Dynamic Tables that refresh incrementally — only new tickets are processed, keeping AI costs proportional to change, not total volume.

---

## Architecture

```
RAW_SUPPORT_TICKETS  (base table — CSV loaded via stage)
  │
  ▼
DT_TICKET_CLASSIFICATION  (Dynamic Table, incremental)
  │  AI_CLASSIFY x3 → product_category, issue_type, priority_bucket
  ▼
DT_TICKET_ENRICHMENT  (Dynamic Table, incremental)
  │  AI_COMPLETE x3 → summary, rationale, manager_recommendation
  ▼
AI_AGG insights  (on-demand, per notebook run)
  ├─ Sarah Chen — top 3 actions for Support
  └─ David Park — top 3 actions for Product
```

| Function | Purpose | Where |
|----------|---------|-------|
| `AI_CLASSIFY` | Fixed-label classification (category, issue, priority) | `DT_TICKET_CLASSIFICATION` |
| `AI_COMPLETE` | Per-row summaries, rationale, recommendations | `DT_TICKET_ENRICHMENT` |
| `AI_AGG` | Cross-row aggregated insights per persona | Notebook cell 20 |

Dynamic Tables use `REFRESH_MODE = INCREMENTAL` and `TARGET_LAG = '1 day'` — Snowflake detects changes and only runs AI functions on the delta.

---

## Quickstart

```bash
# 1. Create database, stage, and load CSV
snowsql -f sql/01_setup.sql

# 2. Open and run the notebook end to end
open notebooks/support_ticket_demo.ipynb
```

---

## Project Layout

```
coco_101/
├── AGENTS.md                     Project context for Cortex Code
├── coco-best-practices.md        CoCo tips: do/avoid, pre-session checklist
├── data/
│   └── support_data.csv          Source ticket data (~2,000 rows)
├── sql/
│   └── 01_setup.sql              DB, schema, stage, table, COPY INTO
├── notebooks/
│   └── support_ticket_demo.ipynb Main demo notebook (23 cells)
└── docs/
    ├── company-brief.md          SnowCare storyline and personas
    ├── taxonomy.md               Allowed classification labels
    ├── prompts.md                Prompt templates
    ├── pipeline.md               Pipeline spec and rules
    └── pipeline-plan.md          Detailed build plan with diagrams
```

---

## Mental Model: What Each File Does

```
AGENTS.md ──────── Project memory    CoCo reads this first every session.
                                     Stack, rules, references — the "briefing."

docs/ ──────────── Constraints       What the pipeline must do and must not do.
                                     Pipeline spec, company brief, prompt templates.

docs/taxonomy.md ─ Stability         The fixed label set. Changes here ripple
                                     through classification, charts, and validation.

notebooks/ ─────── Execution         The runnable demo. Creates DTs, validates,
                                     charts, and generates AI_AGG insights.
```

In short:

- **AGENTS.md** = project memory (CoCo starts here)
- **docs/** = constraints (what and why)
- **taxonomy.md** = stability (change labels, break downstream)
- **notebooks/** = execution (the thing you run)

---

## Building This Project with Cortex Code

This repo was built end-to-end with Cortex Code (CoCo). Here are the practices that made it work well.

### AGENTS.md as Project Memory

CoCo is stateless — each session starts fresh. `AGENTS.md` gives it instant context: the stack, folder layout, coding rules, AI function guidelines, and references. Every session reads it first, so CoCo never starts cold.

### Plan Before You Build

Use `/plan` for anything multi-step. CoCo proposes, you approve, then it executes one phase at a time. The pipeline plan lives at `docs/pipeline-plan.md` — it was the blueprint for every notebook cell.

### Review Every Diff

Always run `git diff` before committing. CoCo does this automatically when you ask it to commit: it checks status, reads the diff, and writes a commit message grounded in the actual changes. This catches accidental edits, stale code, and files that shouldn't be tracked (like `.snowflake/`).

### Right Function for the Job

| Task | Function | Rule |
|------|----------|------|
| Fixed-label outputs | `AI_CLASSIFY` | Labels from taxonomy only |
| Per-row text generation | `AI_COMPLETE` | Summaries, rationale, recommendations |
| Cross-row aggregation | `AI_AGG` | Instruction must be a string constant |

Document these rules in AGENTS.md so CoCo follows them automatically.

### Incremental Dynamic Tables

Since Sep 2025, Snowflake supports AI functions in incremental refresh mode. Only new/changed rows run through AI — this keeps costs proportional to change, not total volume.

### Snowpark Over Connector

Use `session.sql(...).collect()` for DDL/DML and `.to_pandas()` for queries that feed charts. Push compute to Snowflake; only pull aggregated results to Python.

### Cost Awareness

Estimate credits before running AI calls (the notebook has markdown cost-estimate cells). Monitor consumption via `CORTEX_AI_FUNCTIONS_USAGE_HISTORY`.

### Start Fresh When Needed

Sessions have a natural lifespan. When context gets long or CoCo starts looping, save progress in memory and start a new session with a handoff summary.

---

## References

- [AI_CLASSIFY](https://docs.snowflake.com/en/sql-reference/functions/ai_classify)
- [AI_COMPLETE](https://docs.snowflake.com/en/sql-reference/functions/complete-snowflake-cortex)
- [AI_AGG](https://docs.snowflake.com/en/sql-reference/functions/ai_agg)
- [Dynamic Tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about)
- [Cortex Code CLI](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli)
- [Cortex Code Best Practices HoL](https://www.snowflake.com/en/developers/guides/best-practices-cortex-code-cli/)
