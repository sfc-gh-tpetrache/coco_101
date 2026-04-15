# Pipeline Overview

## Goal
Analyze support tickets in Snowflake by classifying them into product categories, issue types, and priority, then enriching them with summaries and insights for PMs and Support Managers.

## Flow

1. `RAW_SUPPORT_TICKETS`
   - Raw input data

2. `DT_TICKET_CLASSIFICATION` (Dynamic Table)
   - Uses `AI_CLASSIFY`
   - Outputs:
     - product_category
     - issue_type
     - priority_bucket

3. `DT_TICKET_ENRICHMENT` (Dynamic Table)
   - Uses `AI_COMPLETE`
   - Inputs: raw ticket + classification outputs
   - Outputs:
     - summary_text
     - rationale
     - manager_recommendation

## Refresh
- Dynamic Tables refresh automatically
- Data should be **at least daily fresh**
- Shorter `target_lag` can be used for demos

## Rules
- Use `AI_CLASSIFY` only for labels
- Use `AI_COMPLETE` only for summaries and insights
- Do not classify the same field twice
- Keep raw data separate from derived outputs

## Validation
- Row counts match across stages
- Labels stay within taxonomy
- Key fields (summary, rationale) are populated
- Spot check sample records

## Notebook Role
The notebook is the main demo:
- Query dynamic tables directly
- Validate outputs
- Show sample records
- Build charts

## Key Outputs

### PM
- Tickets by product_category
- Issue trends over time

### Support Manager
- Tickets by priority_bucket
- High-priority (P0/P1) trends