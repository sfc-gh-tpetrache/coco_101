# Plan: SQL Data-Loading + Notebook Pipeline

## Context

The project has reference docs ([docs/pipeline.md](pipeline.md), [docs/taxonomy.md](taxonomy.md), [docs/prompts.md](prompts.md)) and a CSV at [data/support_data.csv](../data/support_data.csv). The `sql/` and `notebooks/` folders are currently empty.

Per the user's direction:
- **SQL folder** contains only the setup/load script.
- **Notebook** contains everything else: Dynamic Table creation, validation, charts, sample enrichment outputs.
- **Connection**: `oregon_tp` (account: SNOWHOUSE, role: PUBLIC, warehouse: SNOWADHOC -- pipeline SQL will explicitly USE WAREHOUSE AI_WH).
- **Plan artifact**: saved to [docs/pipeline-plan.md](pipeline-plan.md) alongside existing project docs.

### AI Function Skill Best Practices Applied

From the `cortex-ai-functions` skill references:

| Best Practice | How Applied |
|---------------|-------------|
| Display cost estimate before execution | Notebook includes a markdown cell with token/credit estimates before running the DT creation cells |
| Use `task_description` in AI_CLASSIFY config | All three AI_CLASSIFY calls include a `task_description` string for context |
| Use label `description` objects for ambiguous categories | Categories like "Other", "Unknown", "UI/UX" get description objects |
| Use `:labels[0]::VARCHAR` to extract single label | All AI_CLASSIFY outputs use this pattern |
| Require `SNOWFLAKE.CORTEX_USER` role | Setup SQL includes the GRANT statement |

### Dynamic Tables + AI Functions Compatibility

Since [Sep 2025](https://docs.snowflake.com/en/release-notes/2025/other/2025-09-11-dynamic-tables-cortex-aisql-support), Snowflake supports Cortex AI functions in **incremental** dynamic table refresh mode. Both DTs use `REFRESH_MODE = INCREMENTAL` so only new/changed rows are processed on subsequent refreshes, significantly reducing AI function costs after the initial load.

---

## File 1: sql/01_setup.sql

**Purpose:** Create infrastructure and load raw data. This is the only file in `sql/`.

```sql
USE WAREHOUSE AI_WH;

CREATE DATABASE IF NOT EXISTS SNOWCARE_DEMO;
CREATE SCHEMA IF NOT EXISTS SNOWCARE_DEMO.SUPPORT;
USE SCHEMA SNOWCARE_DEMO.SUPPORT;

-- Grant Cortex AI function access (required by AI_CLASSIFY / AI_COMPLETE)
GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE PUBLIC;

-- Internal stage for CSV upload
CREATE OR REPLACE STAGE SUPPORT_STAGE
  DIRECTORY = (ENABLE = TRUE)
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

-- Raw table matching CSV columns exactly
CREATE OR REPLACE TABLE RAW_SUPPORT_TICKETS (
    TICKET_ID                   VARCHAR,
    SUBMIT_DATE                 DATE,
    SUBMIT_TIMESTAMP            TIMESTAMP_NTZ,
    CUSTOMER_ID                 VARCHAR,
    CUSTOMER_TIER               VARCHAR,
    CHANNEL                     VARCHAR,
    PRIORITY                    VARCHAR,
    STATUS                      VARCHAR,
    PRODUCT_AREA                VARCHAR,
    CLASSIFICATION              VARCHAR,
    SENTIMENT                   VARCHAR,
    FIRST_RESPONSE_TIME_HOURS   NUMBER(10,2),
    RESOLUTION_TIME_HOURS       NUMBER(10,2),
    TICKET_SUBJECT              VARCHAR,
    TICKET_DESCRIPTION          TEXT,
    AGENT_NOTES                 TEXT
);

PUT file:///Users/tpetrache/dev/coco_101/data/support_data.csv @SUPPORT_STAGE AUTO_COMPRESS=TRUE OVERWRITE=TRUE;

COPY INTO RAW_SUPPORT_TICKETS
  FROM @SUPPORT_STAGE/support_data.csv
  FILE_FORMAT = (
    TYPE = 'CSV'
    SKIP_HEADER = 1
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    ESCAPE_UNENCLOSED_FIELD = NONE
  )
  ON_ERROR = 'CONTINUE';
```

---

## File 2: notebooks/support_ticket_demo.ipynb

**Purpose:** Main demo artifact. Uses Snowpark Session and Python (pandas + matplotlib) to build the pipeline, validate it, and visualize results.

### Cell-by-cell structure

#### Section A: Setup and Data Load

**Cell 1 (markdown):** Title and overview

```markdown
# Support Ticket Intelligence Demo

End-to-end pipeline that classifies and enriches support tickets using Snowflake AI functions
(AI_CLASSIFY, AI_COMPLETE) via Dynamic Tables.

See [docs/pipeline.md](../docs/pipeline.md) for the pipeline specification.
```

**Cell 2 (code):** Connect to Snowflake

```python
import os
import snowflake.connector
import pandas as pd
import matplotlib.pyplot as plt

conn = snowflake.connector.connect(
    connection_name=os.getenv("SNOWFLAKE_CONNECTION_NAME") or "oregon_tp"
)
cur = conn.cursor()
cur.execute("USE WAREHOUSE AI_WH")
cur.execute("USE SCHEMA SNOWCARE_DEMO.SUPPORT")
print("Connected to SNOWCARE_DEMO.SUPPORT")
```

**Cell 3 (code):** PUT local CSV to stage + COPY INTO raw table

```python
import pathlib
csv_path = pathlib.Path(__file__).resolve().parent.parent / "data" / "support_data.csv"
# Fallback for notebook environments where __file__ is not defined
if not csv_path.exists():
    csv_path = pathlib.Path("../data/support_data.csv").resolve()

cur.execute(f"PUT 'file://{csv_path}' @SUPPORT_STAGE AUTO_COMPRESS=TRUE OVERWRITE=TRUE")

cur.execute("""
COPY INTO RAW_SUPPORT_TICKETS
  FROM @SUPPORT_STAGE/support_data.csv
  FILE_FORMAT = (
    TYPE = 'CSV'
    SKIP_HEADER = 1
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    ESCAPE_UNENCLOSED_FIELD = NONE
  )
  ON_ERROR = 'CONTINUE'
""")

cur.execute("SELECT COUNT(*) FROM RAW_SUPPORT_TICKETS")
print(f"Loaded {cur.fetchone()[0]} rows into RAW_SUPPORT_TICKETS")
```

#### Section B: Classification Dynamic Table

**Cell 4 (markdown):** Cost estimate

```markdown
## Cost Estimate: AI_CLASSIFY

Based on ~2,000 tickets, ~500 tokens avg per ticket, 3 AI_CLASSIFY calls per ticket:
- **Input tokens**: ~3M total -> ~4.5 credits
- **Output tokens**: minimal (label string only)

> Note: Since Sep 2025, Cortex AI functions support **incremental refresh** in Dynamic Tables.
> With `REFRESH_MODE = INCREMENTAL`, only new/changed rows are processed on subsequent refreshes —
> significantly reducing cost after the initial load.
```

**Cell 5 (code):** Create DT_TICKET_CLASSIFICATION

```python
cur.execute("""
CREATE OR REPLACE DYNAMIC TABLE DT_TICKET_CLASSIFICATION
  WAREHOUSE = AI_WH
  TARGET_LAG = '1 day'
  REFRESH_MODE = INCREMENTAL
AS
SELECT
    t.*,
    AI_CLASSIFY(
        t.TICKET_DESCRIPTION,
        [
          {'label':'Billing',        'description':'Invoicing, charges, refunds, subscription changes'},
          {'label':'Authentication', 'description':'Login, SSO, password, session, permissions'},
          {'label':'Performance',    'description':'Slowness, timeouts, latency, degradation'},
          {'label':'UI/UX',          'description':'Dashboard display, usability, interface issues'},
          {'label':'Integrations',   'description':'API connectors, data sync, third-party tools'},
          {'label':'Data Platform',  'description':'Query engine, data pipeline, analytics processing'},
          {'label':'Security',       'description':'Encryption, compliance, vulnerability, access control'},
          {'label':'Other',          'description':'Does not fit any other category'}
        ],
        {'task_description': 'Classify this support ticket into the product category it belongs to'}
    ):labels[0]::VARCHAR AS PRODUCT_CATEGORY,

    AI_CLASSIFY(
        t.TICKET_DESCRIPTION,
        [
          {'label':'Bug',                 'description':'Software defect or broken functionality'},
          {'label':'Feature Request',     'description':'Request for new capability or enhancement'},
          {'label':'Configuration Issue', 'description':'Setup, settings, or config problem'},
          {'label':'Data Issue',          'description':'Wrong data, missing data, data corruption'},
          {'label':'Usage Question',      'description':'How-to question about using the product'},
          {'label':'Access Problem',      'description':'Cannot access a resource due to permissions'},
          {'label':'Unknown',             'description':'Cannot determine the issue type'}
        ],
        {'task_description': 'Classify this support ticket by the type of issue reported'}
    ):labels[0]::VARCHAR AS ISSUE_TYPE,

    AI_CLASSIFY(
        t.TICKET_DESCRIPTION,
        [
          {'label':'P0', 'description':'Critical outage affecting production or revenue'},
          {'label':'P1', 'description':'Major issue with significant business impact'},
          {'label':'P2', 'description':'Moderate issue with workaround available'},
          {'label':'P3', 'description':'Minor issue or cosmetic problem'}
        ],
        {'task_description': 'Assess the urgency of this support ticket based on business impact described'}
    ):labels[0]::VARCHAR AS PRIORITY_BUCKET
FROM RAW_SUPPORT_TICKETS t
""")
print("DT_TICKET_CLASSIFICATION created")
```

#### Section C: Enrichment Dynamic Table

**Cell 6 (markdown):** Cost estimate

```markdown
## Cost Estimate: AI_COMPLETE

Based on ~2,000 tickets, 3 AI_COMPLETE calls per ticket (claude-4-sonnet):
- **Input tokens**: ~3M -> ~4.5 credits
- **Output tokens**: ~600K -> ~4.5 credits
- **Total estimate**: ~9 credits for initial enrichment (incremental refreshes process only new rows)
```

**Cell 7 (code):** Create DT_TICKET_ENRICHMENT

```python
cur.execute("""
CREATE OR REPLACE DYNAMIC TABLE DT_TICKET_ENRICHMENT
  WAREHOUSE = AI_WH
  TARGET_LAG = '1 day'
  REFRESH_MODE = INCREMENTAL
AS
SELECT
    c.*,
    AI_COMPLETE(
        model => 'claude-4-sonnet',
        prompt => 'Summarize this support ticket in one sentence:\\n\\n' || c.TICKET_DESCRIPTION
    ) AS SUMMARY_TEXT,
    AI_COMPLETE(
        model => 'claude-4-sonnet',
        prompt => 'Explain briefly why this support ticket was classified as:\\n\\n'
            || 'product_category: ' || c.PRODUCT_CATEGORY || '\\n'
            || 'issue_type: '       || c.ISSUE_TYPE       || '\\n'
            || 'priority_bucket: '  || c.PRIORITY_BUCKET  || '\\n\\n'
            || 'Ticket:\\n' || c.TICKET_DESCRIPTION
    ) AS RATIONALE,
    AI_COMPLETE(
        model => 'claude-4-sonnet',
        prompt => 'Given this support ticket, suggest the next best action for a support manager:\\n\\n'
            || c.TICKET_DESCRIPTION
    ) AS MANAGER_RECOMMENDATION
FROM DT_TICKET_CLASSIFICATION c
""")
print("DT_TICKET_ENRICHMENT created")
```

#### Section D: Validation

**Cell 8 (markdown):**

```markdown
## Validation

Checks from [docs/pipeline.md](../docs/pipeline.md):
- Row counts match across pipeline stages
- Labels stay within taxonomy
- Key enrichment fields are populated
- Spot check sample records
```

**Cell 9 (code):** Row count comparison

```python
for table in ['RAW_SUPPORT_TICKETS', 'DT_TICKET_CLASSIFICATION', 'DT_TICKET_ENRICHMENT']:
    cur.execute(f"SELECT COUNT(*) FROM {table}")
    print(f"{table}: {cur.fetchone()[0]} rows")
```

**Cell 10 (code):** Label coverage

```python
df_labels = pd.read_sql("""
    SELECT PRODUCT_CATEGORY, ISSUE_TYPE, PRIORITY_BUCKET, COUNT(*) AS CNT
    FROM DT_TICKET_CLASSIFICATION
    GROUP BY ALL ORDER BY CNT DESC
""", conn)
display(df_labels)
```

**Cell 11 (code):** Null check on enrichment columns

```python
df_nulls = pd.read_sql("""
    SELECT
        COUNT_IF(SUMMARY_TEXT IS NULL) AS null_summary,
        COUNT_IF(RATIONALE IS NULL) AS null_rationale,
        COUNT_IF(MANAGER_RECOMMENDATION IS NULL) AS null_recommendation,
        COUNT(*) AS total
    FROM DT_TICKET_ENRICHMENT
""", conn)
display(df_nulls)
```

#### Section E: PM View

**Cell 12 (markdown):**

```markdown
## PM View: Product Category and Issue Trends
```

**Cell 13 (code):** Bar chart -- tickets by product_category

```python
df_cat = pd.read_sql("""
    SELECT PRODUCT_CATEGORY, COUNT(*) AS TICKET_COUNT
    FROM DT_TICKET_CLASSIFICATION
    GROUP BY PRODUCT_CATEGORY ORDER BY TICKET_COUNT DESC
""", conn)
df_cat.plot.barh(x='PRODUCT_CATEGORY', y='TICKET_COUNT', legend=False)
plt.title('Tickets by Product Category')
plt.xlabel('Count')
plt.tight_layout()
plt.show()
```

**Cell 14 (code):** Line chart -- issue trends over time (weekly)

```python
df_trend = pd.read_sql("""
    SELECT DATE_TRUNC('WEEK', SUBMIT_DATE) AS WEEK, PRODUCT_CATEGORY, COUNT(*) AS CNT
    FROM DT_TICKET_CLASSIFICATION
    GROUP BY ALL ORDER BY WEEK
""", conn)
df_trend.pivot(index='WEEK', columns='PRODUCT_CATEGORY', values='CNT').plot()
plt.title('Issue Trends Over Time')
plt.ylabel('Ticket Count')
plt.tight_layout()
plt.show()
```

#### Section F: Support Manager View

**Cell 15 (markdown):**

```markdown
## Support Manager View: Priority Distribution and P0/P1 Trends
```

**Cell 16 (code):** Bar chart -- tickets by priority_bucket

```python
df_pri = pd.read_sql("""
    SELECT PRIORITY_BUCKET, COUNT(*) AS TICKET_COUNT
    FROM DT_TICKET_CLASSIFICATION
    GROUP BY PRIORITY_BUCKET ORDER BY PRIORITY_BUCKET
""", conn)
df_pri.plot.bar(x='PRIORITY_BUCKET', y='TICKET_COUNT', legend=False)
plt.title('Tickets by Priority Bucket')
plt.ylabel('Count')
plt.tight_layout()
plt.show()
```

**Cell 17 (code):** Line chart -- P0/P1 high-priority trends over time

```python
df_high = pd.read_sql("""
    SELECT DATE_TRUNC('WEEK', SUBMIT_DATE) AS WEEK, PRIORITY_BUCKET, COUNT(*) AS CNT
    FROM DT_TICKET_CLASSIFICATION
    WHERE PRIORITY_BUCKET IN ('P0','P1')
    GROUP BY ALL ORDER BY WEEK
""", conn)
df_high.pivot(index='WEEK', columns='PRIORITY_BUCKET', values='CNT').plot()
plt.title('P0/P1 Trends Over Time')
plt.ylabel('Ticket Count')
plt.tight_layout()
plt.show()
```

#### Section G: Sample Enrichment Outputs

**Cell 18 (markdown):**

```markdown
## Sample Enriched Records
```

**Cell 19 (code):** Display 5 enriched tickets

```python
df_samples = pd.read_sql("""
    SELECT TICKET_ID, TICKET_SUBJECT, PRODUCT_CATEGORY, ISSUE_TYPE, PRIORITY_BUCKET,
           SUMMARY_TEXT, RATIONALE, MANAGER_RECOMMENDATION
    FROM DT_TICKET_ENRICHMENT
    LIMIT 5
""", conn)
for _, row in df_samples.iterrows():
    print(f"--- {row['TICKET_ID']}: {row['TICKET_SUBJECT']} ---")
    print(f"Classification: {row['PRODUCT_CATEGORY']} / {row['ISSUE_TYPE']} / {row['PRIORITY_BUCKET']}")
    print(f"Summary: {row['SUMMARY_TEXT']}")
    print(f"Rationale: {row['RATIONALE']}")
    print(f"Recommendation: {row['MANAGER_RECOMMENDATION']}\n")
```

#### Section H: AI-Aggregated Insights (AI_AGG)

**Cell 20 (markdown):**

```markdown
## AI-Generated Insights: Next Actions

Uses **AI_AGG** to generate the top 3 actionable insights for the **Support Manager** and **Product Manager** by aggregating enriched ticket data directly in Snowflake.
```

**Cell 21 (code):** Persona-specific AI_AGG insights

Runs two `AI_AGG` calls over `DT_TICKET_ENRICHMENT`, one per persona. Each aggregates a concatenated ticket-context expression (priority, category, issue type, subject, summary) with a persona-tailored constant instruction.

- **Sarah Chen (Support Manager):** Focuses on what's breaking now, upset customers, escalation patterns, recurring issues, and Monday-morning action items.
- **David Park (Product Manager):** Focuses on feature requests, competitor signals, sprint prioritization evidence (volume, tier, urgency), and cross-module issues.

```python
ticket_expr = """
    'Priority: ' || PRIORITY_BUCKET
    || ' | Category: ' || PRODUCT_CATEGORY
    || ' | Issue: '    || ISSUE_TYPE
    || ' | Subject: '  || TICKET_SUBJECT
    || ' | Summary: '  || SUMMARY_TEXT
"""

personas = {
    "Sarah Chen — VP of Customer Support": (
        "You are Sarah Chen, VP of Customer Support at a B2B SaaS company. "
        "You manage 12 agents across 3 time zones. Your top priorities are: "
        "spotting what is breaking right now, identifying which customers are upset, "
        "reducing escalations, improving first-contact resolution, and catching recurring "
        "patterns early — not months later in a quarterly deck. "
        "Analyze the support tickets below and return exactly 3 concise, actionable insights "
        "ranked by urgency. For each insight include: "
        "(1) a short title, (2) evidence from the tickets, and (3) a concrete next step "
        "you would take Monday morning to protect customers this week."
    ),
    "David Park — Director of Product": (
        "You are David Park, Director of Product at a B2B SaaS company. "
        "You own the roadmap for the Iceberg Billing and Drift User Management modules. "
        "Your top priorities are: surfacing feature requests buried in tickets, "
        "detecting competitor mentions, building evidence for sprint prioritization "
        "(how many customers asked, which tier, how urgent), and spotting cross-module issues. "
        "Analyze the support tickets below and return exactly 3 concise, actionable insights "
        "ranked by product impact. For each insight include: "
        "(1) a short title, (2) evidence from the tickets, and (3) a concrete next step "
        "you would bring to your next sprint planning meeting."
    ),
}

for persona, instruction in personas.items():
    result = session.sql(f"""
        SELECT AI_AGG(
            {ticket_expr},
            '{instruction}'
        ) AS top_insights
        FROM DT_TICKET_ENRICHMENT
    """).collect()[0]["TOP_INSIGHTS"]

    print(f"{'='*60}")
    print(f"  {persona}")
    print(f"{'='*60}")
    print(result)
    print()
```

#### Section I: Cost Monitoring

**Cell 22 (markdown):**

```markdown
## AI Function Cost Monitoring

Uses `SNOWFLAKE.ACCOUNT_USAGE.CORTEX_AI_FUNCTIONS_USAGE_HISTORY` to track credits consumed by AI_CLASSIFY, AI_COMPLETE, and AI_AGG calls. Data has up to 60 min latency; running queries are updated every ~30 min. Re-run this cell after the Dynamic Tables have refreshed.
```

**Cell 23 (code):** Cost monitoring query

```python
df_cost = session.sql("""
    SELECT
        DATE_TRUNC('day', START_TIME) AS USAGE_DATE,
        FUNCTION_NAME,
        MODEL_NAME,
        SUM(CREDITS) AS TOTAL_CREDITS,
        COUNT(DISTINCT QUERY_ID) AS QUERY_COUNT
    FROM SNOWFLAKE.ACCOUNT_USAGE.CORTEX_AI_FUNCTIONS_USAGE_HISTORY
    WHERE START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP())
    GROUP BY 1, 2, 3
    ORDER BY USAGE_DATE DESC, TOTAL_CREDITS DESC
""").to_pandas()
display(df_cost)
print(f"\nTotal AI function credits (last 7 days): {df_cost['TOTAL_CREDITS'].sum():.4f}")
```

---

## Continuous Refresh Pipeline

```
  NEW TICKETS                  DYNAMIC TABLES (auto-refresh, incremental)                INSIGHTS (on demand)
 ─────────────     ┌──────────────────────────────────────────────────┐     ┌──────────────────────────────┐
                   │                                                  │     │                              │
  CSV / App DB     │  DT_TICKET_CLASSIFICATION    DT_TICKET_ENRICHMENT│     │  AI_AGG ─► Support Manager  │
      │            │     AI_CLASSIFY x3               AI_COMPLETE x3  │     │  AI_AGG ─► Product Manager  │
      ▼            │          │                           │           │     │  Charts & Validation         │
 @SUPPORT_STAGE    │          │   new/changed rows        │           │     │                              │
      │            │          ▼ ─────────────────► ▼      │           │     └──────────────────────────────┘
      ▼            │                                      │           │                  ▲
 RAW_SUPPORT  ────►│          only delta rows processed   │           │                  │
  _TICKETS         │          (saves AI credits)          │──────────────────────────────►│
                   │                                                  │
                   └──────────────────────────────────────────────────┘

  ◄── TARGET_LAG = 1 day ── Snowflake detects changes and cascades automatically ──►
```

**How it stays fresh:**
1. New tickets land in `RAW_SUPPORT_TICKETS` (via COPY INTO, Snowpipe, or manual load).
2. Snowflake detects changes and incrementally refreshes each Dynamic Table — only the delta runs through AI functions.
3. `AI_AGG` and charts always query the latest enriched data on every notebook run.

---

## Rules Cross-Check

| Rule (from pipeline.md / taxonomy.md) | How Met |
|----------------------------------------|---------|
| AI_CLASSIFY only for labels | Cell 4 uses AI_CLASSIFY for product_category, issue_type, priority_bucket |
| AI_COMPLETE only for per-row summaries/rationales | Cell 6 uses AI_COMPLETE for summary_text, rationale, manager_recommendation |
| AI_AGG for cross-row aggregation insights | Cell 20 uses AI_AGG to generate persona-specific insights across all tickets |
| AI_AGG instruction must be a string constant | Each persona gets its own AI_AGG call with a literal instruction (no column refs) |
| Do not classify same field twice | Each field has exactly one AI_CLASSIFY call |
| Keep raw data separate from derived | RAW_SUPPORT_TICKETS untouched; derived columns in downstream DTs |
| Labels within taxonomy | Label arrays match docs/taxonomy.md exactly |
| Row counts match across stages | Cell 8 validates |
| Key fields populated | Cell 10 checks for NULLs |
| Spot check sample records | Cells 9 + 18 |
| Cost estimate before AI calls (skill) | Markdown cells 3 + 5 show estimates |
| task_description for AI_CLASSIFY (skill) | All 3 AI_CLASSIFY calls include task_description |
| Label descriptions for ambiguous categories (skill) | All categories use label/description objects |
| CORTEX_USER role required (skill) | Setup SQL includes GRANT statement |
