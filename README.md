# n8n Automation Workflows

AI-powered automation workflows built with n8n by Chris Arnold — AI Workflow Architect in training.

## Personal Automations

### Roast Bot
A fun AI-powered chatbot that lets people ask questions about me and get roast-style responses.
- **Trigger:** Chat message
- **Flow:** Chat Trigger → AI Agent (Claude) + Simple Memory
- **What it does:** Responds to questions about the creator (me) with funny, personalized jokes — roasting myself based on information the bot already knows about me

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

### Error Alert System *(Internal Infrastructure)*
Account-wide error monitoring for all workflows.
- **Trigger:** Any workflow error across the entire n8n account
- **Flow:** Error Trigger → Gmail
- **What it does:** Instantly emails Chris when any workflow fails — includes workflow name, failed node, error message, timestamp (Eastern), and a direct link to the execution

---

## Client Simulations

### Arnold Towing — Customer Inquiry Responder *(Fictional Client)*
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

### Lead Enrichment Bot *(Fictional Client)*
Automated lead research and personalized outreach.
- **Trigger:** Webhook with lead name + email
- **Flow:** Webhook → HTTP Request (Hunter.io API) → AI Agent (Claude) → Gmail
- **What it does:** Enriches lead data with company info, Claude writes a personalized outreach email, sends it automatically
- **Error handling:** Hunter.io failures route to a dedicated Gmail alert instead of crashing the workflow (Level 2 error output pin)

### Okafor Property Management — Triage System *(Fictional Client)*
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

### Priya Nambiar — Lead Validator *(Fictional Client)*

Deterministic lead validation workflow that filters bad form submissions before they hit Google Sheets.

- **Trigger:** Webhook (POST) — name, business name, email, phone, monthly revenue
- **Flow:** Webhook → Code Node (validate) → IF → two paths
- **Valid path:** Google Sheets (Append Row) → Gmail success notification
- **Invalid path:** Gmail rejection notification with specific field list
- **Error path:** Google Sheets Error Output Pin → Gmail alert on sheet write failure
- **What it does:** Checks all 5 fields for presence, validates email format (regex), phone length (10 digits), and revenue type (numeric). Clean leads go to the sheet. Bad ones are rejected with a named field list. Sheet write failures trigger an alert — nothing fails silently.
- **Architectural decision:** Validation uses a Code node, not an LLM. Rules are 100% deterministic — no AI needed, no cost, no non-determinism.
- **Security:** Webhook secured with Header Auth — requests without the correct `x-api-key` header are rejected before any workflow logic runs
- **Error handling:** Error Output Pin on the Google Sheets node — confirmed via testing with a broken spreadsheet ID
- **Built:** April 16, 2026 | **Updated:** April 16, 2026 — webhook auth added

### Jerome Atkins — Lawn Care Request Tracker *(Fictional Client)*
Stateful customer tracking system for a one-man lawn care operation.
- **Trigger:** Webhook (POST) — customer name, phone, job description
- **Flow:** Webhook → Normalize Phone (Code) → SQLite Node (init) → SQLite SELECT → Check Result (Code) → IF → two paths
- **New customer path:** SQLite INSERT → AI Agent (Claude) → Gmail
- **Returning customer path:** AI Agent (Claude) → SQLite UPDATE → Gmail
- **Error path:** SQLite SELECT error pin → Gmail plain alert
- **What it does:** Normalizes phone numbers, looks up the customer in SQLite, and delivers Jerome a scannable briefing before he calls back — new customers get a first-contact summary, returning customers get their full job history
- **Stateful:** Customer records persist in SQLite across all workflow runs. History appends on every new request.
- **AI security:** Both agents include prompt injection defense, system prompt confidentiality, and job description validation — adversarial inputs are rejected with a fallback message
- **Key technical note:** n8n v2.11.4 expression editor silently appends a newline to all expression fields. IF conditions using `equals` always fail because `"yes\n" ≠ "yes"`. Workaround: use `contains` operator which is immune to the trailing character.
- **Community node:** `n8n-nodes-sqlite3` by DangerBlack
- **Built:** Mar 22, 2026
- **Full details:** [jerome-atkins-lawn-care/README.md](jerome-atkins-lawn-care/README.md)

### Sharp & Clean Barbershop — AI Communication System *(Fictional Client)*
Full multi-agent AI communication system for a barbershop — handles FAQs, appointment booking, and customer escalations automatically.
- **Trigger:** n8n Chat Trigger (web chat)
- **Architecture:** Orchestrator agent (Razorback) with Simple Memory + 3 sub-workflow tools
- **Flow:** Chat Trigger → Orchestrator Agent → FAQ / Book / Escalate sub-workflows
- **FAQ sub-workflow:** AI Agent answers questions about hours, services, prices, walk-ins using Tony's business data — no tools, no memory, plain text return
- **Book sub-workflow:** Checks Google Calendar availability via HTTP Request (freebusy API), creates calendar events via HTTP Request (Calendar Events API), sends HTML confirmation email to customer via Gmail
- **Escalate sub-workflow:** Collects customer name + contact info, sends formatted Gmail alert to shop owner, returns confirmation string to orchestrator
- **Double-booking protection:** freebusy check blocks duplicate bookings and returns alternatives
- **Date handling:** Orchestrator system prompt injects current date dynamically via Luxon expression — prevents AI from using training data dates
- **Error handling:** Connected to Error Alert System. All 13 integration tests passed.
- **Key technical note:** n8n v2.9.4 Google Calendar node is broken (sends calendarId as array). Both availability check and event creation use HTTP Request nodes calling the Google Calendar API directly.
- **Built:** Mar 10, 2026
- **Full details:** [sharp-and-clean-barbershop/tonys-sharp-and-clean.md](sharp-and-clean-barbershop/tonys-sharp-and-clean.md)

---

## Infrastructure

### Self-Hosted n8n (Local Docker Sandbox)
Local n8n instance running in Docker — used for learning, testing, and Stage 2 infrastructure practice. Not connected to the n8n Cloud production instance.
- **Stack:** Windows 11 Pro → WSL 2 (Ubuntu) → Docker Desktop → n8n container
- **Access:** `http://localhost:5678`
- **Data persistence:** Docker named volume (`n8n_data`) — survives container restarts
- **Purpose:** Learning sandbox. Production workflows remain on n8n Cloud until a VPS with proper monitoring and recovery plan is in place.
- **Set up:** Mar 13, 2026

---

## Resources

Reference documents covering the technical and economic thinking behind these workflows.

### AI Workflow Economics — Cost Analysis & ROI Framework
Practical cost modeling applied to two production workflows. Covers token measurement, cost-per-run calculations, model selection rationale, and ROI analysis for both one-shot and conversational multi-agent architectures.
- **Full document:** [resources/ai-workflow-economics.md](resources/ai-workflow-economics.md)
- **Workflows analyzed:** Arnold Towing (one-shot responder), Sharp & Clean (conversational multi-agent)

### Workflow Testing Checklist
A reusable pre-deployment checklist applied to every build before it ships. Covers triggers, data flow, AI agent behavior, external integrations, error handling, edge cases, security, and final verification. Includes a known n8n platform gotchas table.
- **Full document:** [resources/workflow-testing-checklist.md](resources/workflow-testing-checklist.md)

---

## Note on System Prompts
System prompts in workflow JSON files are redacted for security. The workflow architecture, node connections, and logic are fully visible.

## Tools Used
- [n8n](https://n8n.io) — Workflow automation
- [Claude](https://anthropic.com) — AI language model
- Gmail — Email delivery
- Google Calendar API — Appointment availability and booking (Sharp & Clean)
- Hunter.io — Lead enrichment API
- Tally — Form trigger
- Tavily — AI search API
- Telegram — Push notifications (Deal Alert System)

## About
Built as part of my transition into AI Workflow Architecture. These are real, working automations — not demos.
