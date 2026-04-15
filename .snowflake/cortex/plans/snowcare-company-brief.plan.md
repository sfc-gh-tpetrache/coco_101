# Plan: SnowCare Company Brief

## Goal

Write `assets/company-brief.md` — a single, self-contained brief that sets the stage for the demo. Mirrors the Budapest meetup brief structure but tailored for support ticket intelligence with two personas.

---

## Company Identity

| Field | Value |
|---|---|
| Name | **SnowCare** |
| Founded | 2021 |
| Product | AI-powered business analytics SaaS platform |
| Customers | ~4,200 B2B accounts across Basic and Enterprise tiers |
| Support volume | ~4,000 tickets/quarter (12,800+ in the CSV dataset) |
| Modules | Iceberg Billing System, Drift User Management, Blizzard Admin Panel, Avalanche Query Engine, Frost Data Pipeline |

---

## Brief Structure

### 1. Company Background
- Who SnowCare is, market position, products, customer base
- Key business metrics (ARR, customers, support volume, NPS)

### 2. Current Support Operations
- How the team works today (manual spreadsheet triage, quarterly decks to product)
- Time spent: ~15 hours/week across the CS team on manual ticket analysis
- Data silos: support sees tickets, product sees JIRA, nobody connects the dots

### 3. Two Personas with Quotes

**Sarah Chen, VP of Customer Support**
- Goal: reduce escalations, spot patterns early, proactive not reactive
- Pain: drowning in tickets, trends only visible in hindsight
- Quote establishing her frustration with manual work

**David Park, Director of Product**
- Goal: data-driven roadmap, understand competitive pressure from tickets, prioritize the right features
- Pain: gets a summary once a quarter, misses urgent signals
- Quote establishing his need for real-time feature/issue intelligence

### 4. Pain Points (Strategic + Operational)
- No automated classification of tickets by product area or issue type
- Competitor mentions buried in free-text — never surfaced
- Feature requests scattered, never aggregated or ranked
- Urgency/priority set by customers, not validated by content analysis
- Reactive reporting cycle (quarterly deck vs. continuous insight)

### 5. POC Requirements — Hybrid AI Approach

**AI_CLASSIFY** (stable labels for dashboards):
- `product_category` — which module the ticket relates to
- `issue_type` — bug, feature_request, billing, performance, security, onboarding
- `priority_bucket` — critical / high / medium / low (content-derived, not customer-set)

**AI_COMPLETE** (rich text for PM/support-manager story):
- `summary` — 2-3 sentence distillation of the ticket
- `root_cause_hypothesis` — what might be causing the issue based on description
- `manager_recommendation` — suggested next action for the support manager

Must Have (P0):
- Hybrid enrichment running on the full ticket backlog
- Natural language query interface for both personas
- Persona-specific dashboard views

Should Have (P1):
- Weekly auto-generated intelligence brief
- Competitor mention tracking with trend line
- Feature request ranking weighted by customer tier (Enterprise 3x)

### 6. Success Criteria + ROI Impact

| Criteria | Target | ROI Impact |
|---|---|---|
| Ticket analysis time | 15 h/week to < 30 min | ~750 analyst hours saved/year (~$37K at $50/h) |
| Time-to-insight for product | Quarterly to real-time | Faster feature prioritization, shorter release cycles |
| Escalation reduction | 18% to < 12% | ~$120K/year in reduced L2/L3 support cost |
| Competitor signal detection | 0 (manual) to automated | Early competitive response — hard to quantify but strategic |
| Feature request capture rate | ~20% (manual tagging) to 95%+ | Better product-market fit, reduced churn |
| Accuracy of AI classification | > 90% vs human labels | Trust threshold for production adoption |

### 7. POC Timeline

| Milestone | Target |
|---|---|
| Data loaded + enrichment running | Session 1 |
| Dashboard for both personas | Session 2 |
| Present to leadership | Day 3 |
| Feedback + iteration | Day 4 |
| Go/no-go decision | Day 5 |

### 8. Why Snowflake + Cortex

- **AI_CLASSIFY** — deterministic labels without model training
- **AI_COMPLETE** — LLM-generated summaries and recommendations in SQL
- **Semantic Views + Cortex Analyst** — natural language queries for non-technical users
- **Full auditability** — SQL behind every answer
- **Single platform** — data, AI, and app layer without moving data out

---

## File Output

```
assets/company-brief.md   ← the deliverable
```

The brief will be ~150-200 lines of markdown, ready to hand to a prospect or use as the opening context slide for the demo.
