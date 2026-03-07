# Okafor Property Management — Triage System

> **Portfolio Project — Fictional client. Built as a training exercise.**

An automated tenant request triage system that receives requests from multiple channels, classifies them using a two-layer AI engine, and routes them to the correct person instantly.

---

## The Problem

Sandra Okafor manages 14 rental properties with a small team. Tenant requests came in across 3 channels (web form, email, SMS) and her office manager manually checked all three and forwarded them — 2-3 hours per day. A tenant submitted an emergency water leak on a Friday evening that nobody saw until Monday morning.

---

## What Was Built

4 workflows total. One shared Triage Engine called by three thin intake workflows.

```
[Web Form Intake]  ─┐
[Email Intake]     ─┤──► [Triage Engine] ──► EMERGENCY → Telegram + Email
[SMS Intake]       ─┘                    ──► MAINTENANCE → Office Manager
                                         ──► BILLING → Sandra
                                         ──► AMBIGUOUS → Sandra (manual review)
```

---

## Workflows

### Okafor — Triage Engine (Sub-Workflow)
The core logic. Called by all three intake workflows.

**Nodes:**
1. `Execute Workflow Trigger` — receives `message`, `tenantName`, `unit`, `channel`
2. `Keyword Filter` (Code) — scans for hard emergency words: fire, gas, leak, flood, smoke, locked out, break-in. Bypasses AI on match.
3. `Triage Classifier` (AI Agent — Claude) — classifies message into one of 4 categories. Single-word output only.
4. `Keyword Match?` (IF) — routes based on whether keyword check already caught it
5. `Extract AI Category` (Code) — normalizes AI output, pulls original tenant data back using `$('Keyword Filter').first().json`
6. `EMERGENCY?` (IF) — fires Telegram + Email simultaneously on True
7. `Alert — Sandra Telegram` — immediate push notification to property owner
8. `Email — Maintenance Emergency` (Gmail) — emergency alert to on-call staff
9. `MAINTENANCE?` (IF) — routes non-emergency maintenance requests
10. `Email — Office Manager` (Gmail) — maintenance scheduling
11. `BILLING?` (IF) — routes billing inquiries
12. `Email — Sandra Billing` (Gmail) — owner handles financial questions
13. `Email — Sandra Ambiguous` (Gmail) — flagged for manual review, no AI summary

**Two-layer triage:**
- Layer 1: Keyword check (deterministic, fast, no AI cost)
- Layer 2: Claude AI (handles ambiguous cases, biased toward safety)

**Flagging transparency:** Every emergency alert explains *why* it was flagged — keyword match or AI judgment call. Builds client trust over time.

---

### Okafor — Web Form Intake
**Nodes:** Webhook (`/okafor-webform`) → Execute Sub-Workflow

---

### Okafor — Email Intake
**Nodes:** Gmail Trigger (polls every 1 min, unread only) → Execute Sub-Workflow

**Note:** Requires a dedicated tenant intake email address in production. Using Sandra's personal inbox creates a loop risk — triage notification emails get re-processed.

---

### Okafor — SMS Intake
**Nodes:** Webhook (`/okafor-sms`) → Execute Sub-Workflow

**Note:** Webhook represents the Twilio POST endpoint. In production, a dedicated Twilio business number sends tenant SMS to this webhook automatically.

---

## Pre-Deployment Requirements (Real Client)

- Dedicated tenant intake email (e.g., `requests@okaforproperties.com`)
- Dedicated Twilio business number (not owner's personal phone)
- Tenants informed of new intake channels
- Keyword list reviewed with client before go-live
- All placeholder email addresses updated to real staff addresses

---

## Error Handling

All 4 workflows connected to the account-wide Error Alert System. Any failure emails Chris instantly with workflow name, failed node, error message, and execution link.

---

## Note on System Prompts

The Triage Classifier system prompt is redacted in this public JSON. Unredacted originals are stored in the private backup folder.
