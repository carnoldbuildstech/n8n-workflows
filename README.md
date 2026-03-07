# n8n Automation Workflows

AI-powered automation workflows built with n8n by Chris Arnold — AI Workflow Architect in training.

## Workflows

### Arnold Towing — Customer Inquiry Responder *(Portfolio Project — Fictional Client)*
Automated customer inquiry system for a towing company.
- **Trigger:** Tally form submission
- **Flow:** Webhook → Code Node → IF-1 → IF-2 → AI Agent (Claude) → 2x Gmail
- **What it does:** Sends an AI-written dispatch alert to the business owner and a professional confirmation email to the customer — automatically, 24/7
- **Conditional logic:** 3-path routing based on time of inquiry:
  - **Business Hours** (Mon–Fri 7am–7pm) → urgent dispatch alert
  - **After Hours** (Mon–Fri outside business hours) → after hours alert, customer contacted next morning
  - **Weekend** (Sat–Sun) → weekend alert, customer contacted Monday
- **Code node:** JavaScript calculates EST time category from UTC timestamp, outputs `timeCategory` field
- **Updated:** Mar 3, 2026 — upgraded from simple IF node to 3-path conditional routing

### Roast Bot
A fun AI-powered chatbot that lets people ask questions about me and get roast-style responses.
- **Trigger:** Chat message
- **Flow:** Chat Trigger → AI Agent (Claude) + Simple Memory
- **What it does:** Responds to questions about the creator (me) with funny, personalized jokes — roasting myself based on information the bot already knows about me

### Lead Enrichment Bot
Automated lead research and personalized outreach.
- **Trigger:** Webhook with lead name + email
- **Flow:** Webhook → HTTP Request (Hunter.io API) → AI Agent (Claude) → Gmail
- **What it does:** Enriches lead data with company info, Claude writes a personalized outreach email, sends it automatically
- **Error handling:** Hunter.io failures route to a dedicated Gmail alert instead of crashing the workflow (Level 2 error output pin)

### Error Alert System
Account-wide error monitoring for all workflows.
- **Trigger:** Any workflow error across the entire n8n account
- **Flow:** Error Trigger → Gmail
- **What it does:** Instantly emails Chris when any workflow fails — includes workflow name, failed node, error message, timestamp (Eastern), and a direct link to the execution

### AI Customer Service Agent
General-purpose AI customer service automation.
- **Trigger:** Webhook
- **Flow:** AI Agent (Claude) handles responses
- **What it does:** Handles customer inquiries with AI

### 3D Print Finder
AI-powered 3D print file search assistant.
- **Trigger:** Chat message (bookmarked URL)
- **Flow:** Chat Trigger → AI Agent (Claude) + Simple Memory + Tavily Search
- **What it does:** User describes what they want to 3D print, Claude searches Printables.com and Thingiverse.com automatically and returns the top matching STL file links — no manual searching required

### Deal Alert System
Personal retail arbitrage deal finder with AI scoring.
- **Trigger:** Slickdeals RSS feed (polls every 60 minutes, 8am–10pm only)
- **Flow:** RSS Feed Trigger → IF (time window) → Code Node (keyword filter) → AI Agent (Claude) → IF (score 7+) → Telegram
- **What it does:** Monitors Slickdeals for clearance deals in target categories, scores each deal 1-10 for resale profit potential on eBay/Facebook Marketplace, and sends a Telegram push notification for any deal scoring 7 or higher
- **Target categories:** LEGO, tools (DeWalt/Milwaukee/Ryobi), electronics, headphones, speakers, toys/action figures, video games (especially Nintendo), small kitchen appliances, PC parts (GPU, CPU, RAM, SSD)
- **Smart filtering:** JavaScript keyword filter eliminates irrelevant deals before AI scoring — saves API calls and n8n executions
- **Error handling:** Connected to Error Alert System for account-wide monitoring
- **Built:** Mar 4, 2026

---

## Portfolio Projects

### Okafor Property Management — Triage System *(Portfolio Project — Fictional Client)*
Multi-channel tenant request triage system for a 14-property management company.
- **Trigger:** 3 intake channels — Web Form (Webhook), Email (Gmail Trigger), SMS (Webhook)
- **Architecture:** 3 thin intake workflows all calling 1 shared Triage Engine sub-workflow
- **Triage Engine:** Two-layer classification — keyword check (deterministic) runs before AI. Keywords bypass Claude entirely for speed and reliability.
- **AI Agent:** Claude classifies non-keyword messages into EMERGENCY, MAINTENANCE, BILLING, or AMBIGUOUS — single word output, biased toward safety
- **4-path routing:**
  - EMERGENCY → Telegram alert to owner + Gmail to maintenance staff (both fire simultaneously)
  - MAINTENANCE → Gmail to office manager
  - BILLING → Gmail to owner
  - AMBIGUOUS → Gmail to owner, flagged for manual review
- **Flagging transparency:** Every emergency alert states why it was flagged — keyword match or AI judgment call
- **Error handling:** Connected to Error Alert System for account-wide monitoring
- **Built:** Mar 7, 2026
- **Full details:** [Okafor Triage Project/README.md](Okafor%20Triage%20Project/README.md)
- **Build report:** [okafor-triage-build-report.md](okafor-triage-build-report.md)

---

## Note on System Prompts
System prompts in workflow JSON files are redacted for security. The workflow architecture, node connections, and logic are fully visible.

## Tools Used
- [n8n](https://n8n.io) — Workflow automation
- [Claude](https://anthropic.com) — AI language model
- Gmail — Email delivery
- Hunter.io — Lead enrichment API
- Tally — Form trigger
- Tavily — AI search API
- Telegram — Push notifications (Deal Alert System)

## About
Built as part of my transition into AI Workflow Architecture. These are real, working automations — not demos.
