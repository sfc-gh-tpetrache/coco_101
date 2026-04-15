---
name: "support-ticket-demo"
created: "2026-04-15T10:31:47.742Z"
status: pending
---

# Demo Plan: Snow InsAIght Support Intelligence

## Fictional Company

**Snow InsAIght Inc.** — a B2B SaaS platform for AI-powered business analytics, already referenced throughout the CSV data (product modules: Iceberg Billing System, Drift User Management, Blizzard Admin Panel). Ticket IDs follow `SP-2025-XXXXX` format, customer tiers: Basic / Enterprise.

---

## Personas

| Persona                  | Name       | Goal                                                                               |
| ------------------------ | ---------- | ---------------------------------------------------------------------------------- |
| Customer Support Manager | Sarah Chen | Identify ticket patterns, track urgency trends, reduce escalations                 |
| Product Owner            | David Park | Prioritize features, understand competitive threats, data-driven roadmap decisions |

**Current pain**: Sarah's team spends \~15 hours/week manually reading tickets in spreadsheets. David gets a summary deck once a quarter — too slow to act on.

---

## Data Schema

From `support_data.csv` (12,880 rows) and `AI_SOL_SUPPORT_SETUP.sql`:

```
TICKET_ID, SUBMIT_DATE, CUSTOMER_ID, CUSTOMER_TIER, CHANNEL,
PRIORITY, STATUS, PRODUCT_AREA, SENTIMENT, CLASSIFICATION,
RESPONSE_TIME, RESOLUTION_TIME, TICKET_DESCRIPTION
```

Known product areas in data: `Iceberg Billing System`, `Drift User Management`, `Blizzard Admin Panel` (+ others).

---

## Architecture

```
flowchart TD
    CSV[support_data.csv on GitHub] -->|COPY FILES| Stage[Snowflake Stage]
    Stage -->|COPY INTO| Raw[RAW_SUPPORT_TICKETS]
    Raw -->|AI_EXTRACT + AI_CLASSIFY + AI_SENTIMENT| Enriched[ENRICHED_TICKETS]
    Enriched -->|AI_AGG| Insights[Aggregate Insight Views]
    Enriched -->|Semantic View| Analyst[Cortex Analyst NL Queries]
    Insights --> App[Streamlit Demo App]
    Analyst --> App
    Insights -->|AI_COMPLETE| Brief[Support Intelligence Brief MD]
```

---

## Step 1: Company Brief (`assets/company-brief.md`)

Mirror the Budapest brief structure:

- **Company Background**: Snow InsAIght Inc., founded 2021, 4,200 customers, B2B SaaS analytics platform
- **Key Metrics**: \~4,000 support tickets/quarter, 72h avg resolution time, 18% escalation rate on Enterprise tier
- **Personas**: Sarah Chen (CS Manager) + David Park (Product Owner) with quotes and goals
- **Pain Points**: Manual analysis, reactive not proactive, no competitive signal tracking, data silos between support and product
- **POC Requirements (P0/P1)**: Automated issue classification, competitor mention detection, feature request extraction, urgency scoring
- **Success Criteria**: Reduce analysis time from 15h to <30min, surface competitor threats weekly, give product team a ranked backlog input

---

## Step 2: Snowflake Setup (`setup.sql`)

```
CREATE DATABASE IF NOT EXISTS SNOW_INSAIGHT_DEMO;
CREATE SCHEMA IF NOT EXISTS SNOW_INSAIGHT_DEMO.SUPPORT;

CREATE STAGE IF NOT EXISTS SNOW_INSAIGHT_DEMO.SUPPORT.support_stage
    DIRECTORY = (ENABLE = true)
    ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

-- Load CSV from GitHub URL
COPY FILES INTO @support_stage
  FROM 'https://raw.githubusercontent.com/sfc-gh-tchristian/AI-Accelerators/main/...'
  ...

CREATE OR REPLACE TABLE RAW_SUPPORT_TICKETS (
    TICKET_ID VARCHAR, SUBMIT_DATE TIMESTAMP_NTZ,
    CUSTOMER_ID VARCHAR, CUSTOMER_TIER VARCHAR,
    CHANNEL VARCHAR, PRIORITY VARCHAR, STATUS VARCHAR,
    PRODUCT_AREA VARCHAR, SENTIMENT VARCHAR,
    CLASSIFICATION VARCHAR, RESPONSE_TIME NUMBER,
    RESOLUTION_TIME NUMBER, TICKET_DESCRIPTION TEXT
);
```

---

## Step 3: AI Enrichment Layer (`enrichment.sql`)

```
CREATE OR REPLACE TABLE ENRICHED_TICKETS AS
SELECT *,
  AI_EXTRACT(TICKET_DESCRIPTION, [
    'product_issue',        -- specific bug or error described
    'feature_request',      -- improvement or new capability asked
    'competitor_mention',   -- other tools/vendors referenced
    'urgency_signal'        -- business impact phrases
  ]) AS extracted,
  AI_CLASSIFY(TICKET_DESCRIPTION,
    ['bug', 'feature_request', 'billing', 'performance', 'security', 'onboarding']
  ) AS issue_type,
  AI_CLASSIFY(TICKET_DESCRIPTION,
    ['critical', 'high', 'medium', 'low']
  ) AS ai_urgency_level
FROM RAW_SUPPORT_TICKETS;
```

---

## Step 4: Aggregate Insights (`insights.sql`)

Two sets of AI\_AGG queries:

**CS Manager (Sarah)**:

- Top 5 recurring product issues this quarter
- Urgency distribution by product area
- Escalation rate trend by customer tier
- Sentiment breakdown by channel

**Product Owner (David)**:

- Top 10 feature requests ranked by frequency
- Competitor mentions with context
- Product area health (issue density vs resolution time)
- "What are customers asking us to build that competitors already offer?"

---

## Step 5: Semantic View (`semantic_view.yaml`)

```
name: SUPPORT_INTELLIGENCE
tables:
  - name: ENRICHED_TICKETS
metrics:
  - name: ticket_count
  - name: escalation_rate
  - name: avg_resolution_hours
dimensions:
  - name: product_area
  - name: customer_tier
  - name: ai_urgency_level
  - name: issue_type
  - name: sentiment
verified_queries:
  - "Which product areas have the most critical tickets?"
  - "What features are customers requesting most often?"
  - "Which competitors are mentioned in negative tickets?"
```

---

## Step 6: Streamlit App (`app.py`)

Two-tab layout:

**Tab 1 — CS Manager (Sarah)**

- KPI cards: total tickets, escalation rate, avg resolution time, % critical
- Bar chart: tickets by product area colored by urgency
- Heatmap: sentiment x product area
- Table: top escalated tickets with AI-extracted urgency signals
- Chat box: ask questions via Cortex Analyst

**Tab 2 — Product Owner (David)**

- Ranked feature request list (AI extracted + frequency scored)
- Competitor mention bubble chart (which tools, how often, in what context)
- Product area health matrix (issue volume vs resolution time quadrant)
- AI-generated weekly brief section using AI\_COMPLETE
- Chat box: "What should I prioritize this sprint?"

---

## Step 7: Generated Brief

AI\_COMPLETE produces a markdown brief with sections:

1. Executive Summary (2-3 bullets)
2. Top Product Issues (by area, with representative ticket quotes)
3. Competitive Signals (competitor names, context, frequency)
4. Feature Prioritization (ranked table with customer tier weighting)
5. Recommended Actions — split by persona (Sarah / David)

---

## File Structure

```
demo/
  assets/
    company-brief.md        ← Step 1
  sql/
    setup.sql               ← Step 2
    enrichment.sql          ← Step 3
    insights.sql            ← Step 4
  semantic_view.yaml        ← Step 5
  app.py                    ← Step 6 (Streamlit)
  README.md
```
