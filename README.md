# n8n Automation Workflows

AI-powered automation workflows built with n8n by Chris Arnold — AI Workflow Architect in training.

## Workflows

### Arnold Towing — Customer Inquiry Responder
Automated customer inquiry system for a towing company.
- **Trigger:** Tally form submission
- **Flow:** Webhook → AI Agent (Claude) → 2x Gmail
- **What it does:** Sends an urgent dispatch alert to the business owner and a professional confirmation email to the customer — automatically, 24/7

### Roast Bot
A fun AI-powered roast generator.
- **Trigger:** Chat message
- **Flow:** Chat Trigger → AI Agent (Claude) + Simple Memory
- **What it does:** Roasts the user based on what they say, remembers the conversation

### Lead Enrichment Bot
Automated lead research and personalized outreach.
- **Trigger:** Webhook with lead name + email
- **Flow:** Webhook → HTTP Request (Hunter.io API) → AI Agent (Claude) → Gmail
- **What it does:** Enriches lead data with company info, Claude writes a personalized outreach email, sends it automatically

### AI Customer Service Agent
General-purpose AI customer service automation.
- **Trigger:** Webhook
- **Flow:** AI Agent (Claude) handles responses
- **What it does:** Handles customer inquiries with AI

## Tools Used
- [n8n](https://n8n.io) — Workflow automation
- [Claude](https://anthropic.com) — AI language model
- Gmail — Email delivery
- Hunter.io — Lead enrichment API
- Tally — Form trigger

## About
Built as part of my transition into AI Workflow Architecture. These are real, working automations — not demos.
