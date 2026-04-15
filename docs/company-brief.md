# Company Background: SnowCare

## About Your Company

**SnowCare** is a fast-growing B2B SaaS platform providing AI-powered business analytics to mid-market and enterprise organizations. Founded in 2021 and headquartered in San Francisco, SnowCare helps companies transform raw data into actionable insights through its modular analytics suite.

### Key Business Metrics
- **Customers:** ~4,200 B2B accounts across 18 industries
- **Annual Recurring Revenue:** $48M ARR
- **Customer Tiers:** Basic (3,400 accounts) and Enterprise (800 accounts)
- **Support Volume:** ~4,000 tickets/quarter (~16,000/year)
- **Average Resolution Time:** 72 hours
- **Escalation Rate:** 18% (Enterprise tier)
- **NPS Score:** 42 (industry avg: 36)
- **Customer Retention:** 91% (target: 95%)

### Product Modules

| Module | Description | Customer Adoption |
|--------|-------------|-------------------|
| Iceberg Billing System | Usage-based billing engine with flexible plan management | 92% of accounts |
| Drift User Management | Role-based access control and user provisioning | 85% of accounts |
| Blizzard Admin Panel | Administrative dashboard for account configuration | 78% of accounts |
| Avalanche Query Engine | Core analytics query engine powering all reports | 100% of accounts |
| Frost Data Pipeline | ETL/ELT pipeline builder for data ingestion | 65% of accounts |

---

## Current Support Operations

### How the Team Works Today

SnowCare's support team of 12 agents handles tickets across email, phone, and chat. When a ticket arrives, an agent reads the full description, manually tags the product area and priority, and routes it to the appropriate specialist. At the end of each quarter, a senior analyst spends 2-3 weeks compiling a summary deck for the product team — aggregating common issues, feature requests, and escalation patterns by hand in spreadsheets.

### The Problem

- **15 hours/week** spent by the CS team manually reading and categorizing tickets
- **Quarterly reporting cycle** means product gets insights 3 months too late
- **Competitor mentions** buried in free-text ticket descriptions are never surfaced
- **Feature requests** are scattered across tickets, Slack, and JIRA — never aggregated or ranked
- **Priority is customer-set**, not validated by content analysis — leading to misrouted tickets
- **No connection** between support data and product roadmap decisions

---

## Key Stakeholders

### Customer Support Manager
**Sarah Chen, VP of Customer Support**
- 8 years in B2B SaaS support, joined SnowCare in 2022
- Manages a team of 12 support agents across 3 time zones
- Focused on reducing escalations and improving first-contact resolution
- Frustrated by the reactive nature of current operations — patterns only visible in hindsight

Key quote:
> "By the time we spot a trend in our tickets, it's already been hurting customers for weeks. I need to see patterns as they emerge — not three months later in a quarterly deck."

**On data and AI:**
> "I need to open something Monday morning and instantly know: what's breaking, who's upset, and where should my team focus this week."

### Product Owner
**David Park, Director of Product**
- Former engineering lead, transitioned to product 3 years ago
- Owns the roadmap for the Iceberg Billing and Drift User Management modules
- Wants to make data-driven prioritization decisions, not rely on gut feel
- Gets competitive intelligence anecdotally from sales calls — wants systematic tracking

Key quote:
> "Our customers tell support things they'd never tell us in a product survey. Feature requests, competitor comparisons, workaround hacks — it's all in the ticket data. We're just not mining it."

**On prioritization:**
> "Every sprint planning, I'm asked 'why this feature over that one?' I need evidence — how many customers asked for it, which tier, how urgent. Right now I'm guessing."

---

## Pain Points Identified by Leadership

### Strategic Challenges
1. **Blind to competitive threats:** Customers mention competing tools in tickets, but no one systematically tracks these signals
2. **Feature requests are invisible:** Support agents close tickets without capturing the underlying product need
3. **Priority mismatch:** Customer-set priority doesn't reflect actual business impact — critical issues sometimes filed as "low"
4. **Quarterly insight lag:** Product team decisions are based on 3-month-old data
5. **No cross-module view:** Issues spanning multiple modules (e.g., billing + user management) fall through the cracks

### Operational Challenges
1. **Manual ticket classification:** Agents spend 5-8 minutes per ticket reading, tagging, and routing
2. **Inconsistent categorization:** Different agents tag the same issue differently
3. **No root-cause analysis:** Tickets are resolved individually without connecting recurring patterns
4. **Escalation is reactive:** L2/L3 escalations happen after customer frustration, not before
5. **Data silos:** Support uses Zendesk, product uses JIRA, leadership uses dashboards — nobody has the full picture

---

## POC Requirements from Leadership

### Must Have (P0)
- [ ] Automated ticket classification by product area and issue type
- [ ] AI-generated ticket summaries replacing manual reading
- [ ] Priority validation — content-derived urgency vs. customer-set priority


### Should Have (P1)
- [ ] Competitor mention detection and tracking over time
- [ ] Root-cause hypothesis generation for recurring issues
- [ ] Manager-specific action recommendations per ticket


## Success Criteria

| Criteria | Current State | Target | ROI Impact |
|----------|---------------|--------|------------|
| Ticket analysis time | 15 h/week manual | < 30 min automated | ~750 analyst hours saved/year (~$37K at $50/h) |
| Time-to-insight for product | Quarterly deck | Real-time dashboard | Faster feature prioritization, ~2 weeks shorter release cycles |
| Escalation rate | 18% | < 12% | ~$120K/year in reduced L2/L3 support cost |
| Competitor signal detection | Manual / ad-hoc | Automated with trends | Early competitive response — estimated 5-8% churn prevention on at-risk accounts |
| Feature request capture rate | ~20% (manual tagging) | 95%+ (AI extraction) | Better product-market fit, estimated 3-5% improvement in retention |
| AI classification accuracy | N/A | > 90% vs. human labels | Trust threshold for production adoption |
| First-contact resolution | 64% | 75%+ | ~$85K/year savings from reduced ticket re-opens and follow-ups |

**Estimated total annual ROI: $250K-$350K** in direct savings plus strategic value from competitive intelligence and product-market fit improvements.


## Your Team

**3 analysts** in business intelligence:
- SQL and Python proficient, familiar with Snowflake
- Currently spend 40% of time on manual ticket categorization and reporting
- Want self-service analytics without waiting for data engineering
- Skeptical that AI classification can match human accuracy on nuanced support tickets
