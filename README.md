# AI Customer Support Automation Platform

**n8n Capstone Project — I3C Summer School 2026 | Assignment 5 (Customer Service domain)**

An end-to-end, AI-powered customer support automation platform built on self-hosted n8n. The platform ingests support tickets from multiple channels, classifies and prioritizes them with AI, assigns them to agents, manages the full communication lifecycle, and closes the loop with feedback collection and analytics — with zero manual ticket routing.

---

## Problem Statement

Support teams at growing companies receive tickets through email, web forms, and chat. Manually categorizing, prioritizing, assigning, and tracking these tickets leads to delayed responses, inconsistent prioritization, duplicate tickets, and no visibility into recurring issues. This platform automates the entire lifecycle while keeping humans in the loop for escalations.

## Architecture

Five independent, interconnected n8n workflows, chained via `Execute Workflow` nodes and a shared Postgres (Supabase) database for state and audit logging.

```
WF1 (Ticket Collection)
   → WF2 (AI Classification & Prioritization)
      → WF3 (Agent Assignment & Escalation) ←→ Cron (SLA breach check)
         → WF4 (Customer Communication & Resolution Tracking)
            → WF5 (Feedback Collection & Support Analytics)
```

See `/docs/architecture-diagram.png` for the full visual diagram.

## Workflows

| # | Workflow | Trigger | Core Function |
|---|---|---|---|
| 1 | Ticket Collection & Registration | Webhook (web form / chatbot) + Gmail Trigger | Normalizes intake, creates/updates customer, inserts ticket, logs event |
| 2 | AI Classification & Prioritization | Execute Workflow (from WF1) | Gemini AI tags category + urgency, updates ticket |
| 3 | Agent Assignment & Escalation | Execute Workflow (from WF2) + Cron (SLA check) | Routes ticket to an agent, escalates on SLA breach |
| 4 | Customer Communication & Resolution Tracking | Execute Workflow (from WF3) + Webhook (status updates) | Sends assignment/status/resolution emails, routes by status, triggers WF5 on resolution |
| 5 | Feedback Collection & Support Analytics | Execute Workflow (from WF4) + Webhook (feedback submit) + Cron (weekly report) | Sends CSAT survey, tags feedback themes, closes ticket, builds weekly manager report |

## Tech Stack

- **Automation:** n8n (self-hosted, Docker)
- **Database:** Supabase (Postgres, Session Pooler)
- **AI:** Google Gemini (gemini-2.5-flash-lite) for classification and summarization
- **Email:** Gmail (OAuth2)
- **Triggers:** Webhook (POST), Gmail Trigger, Cron (scheduled SLA + weekly report)

## Advanced Features Implemented

- AI-powered decision making (ticket classification + priority scoring)
- Conditional branching (priority-based routing, status-based routing)
- Error handling and automatic retry on critical nodes
- Logging and audit trail (every workflow writes to a shared log table)
- Scheduled workflows (Cron: SLA breach checker, weekly analytics report)
- Webhook-triggered workflows (ticket intake, status updates, feedback submission)
- Sub-workflow orchestration (`Execute Workflow` chaining across all 5 workflows)

## Repository Structure

```
/workflows/       → Exported n8n workflow JSON files (WF1–WF5)
/docs/            → Architecture diagram, problem analysis, node-by-node documentation
/presentation/    → Slide deck (PPTX/PDF)
/demo/            → Link to demo video
README.md
```

## Setup

1. Self-hosted n8n instance (Docker) running locally or on a server
2. Supabase project (Postgres) — Session Pooler connection string
3. Google Gemini API credential
4. Gmail OAuth2 credential (scopes: read, compose, send)
5. Import each workflow JSON from `/workflows/` into n8n
6. Re-link each `Execute Workflow` node to the corresponding imported workflow (n8n assigns new IDs on import)
7. Publish all 5 workflows, starting from WF5 up to WF1 (a workflow must be published before anything that calls it can be published)

## Author

Ashutosh Singh Yadav — I3C Summer School 2026, IIT Jammu
