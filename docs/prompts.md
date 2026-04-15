# Prompt Definitions

This file defines prompts used with AI_COMPLETE.

## Core Principle
- AI_CLASSIFY → labels (product_category, issue_type, priority_bucket)
- AI_COMPLETE → summaries, explanations, recommendations
- Do NOT duplicate classification logic in AI_COMPLETE

---

## Prompt: Summary

Purpose:
Provide a quick understanding of a support ticket.

Output:
- one concise sentence
- no hallucinated details

Template:

Summarize this support ticket in one sentence:

{{ticket_text}}

---

## Prompt: Rationale

Purpose:
Explain why a ticket was classified a certain way.

Inputs:
- ticket_text
- product_category
- issue_type
- priority_bucket

Output:
- short explanation grounded in the text

Template:

Explain briefly why this support ticket was classified as:

product_category: {{product_category}}
issue_type: {{issue_type}}
priority_bucket: {{priority_bucket}}

Ticket:
{{ticket_text}}

---

## Prompt: Manager Recommendation

Purpose:
Suggest next action for a support manager.

Output:
- short actionable recommendation

Template:

Given this support ticket, suggest the next best action for a support manager:

{{ticket_text}}