# AI Workflow Economics — Cost Analysis & ROI Framework

**Author:** Chris Arnold
**Date:** March 2026
**Purpose:** Demonstrate practical understanding of AI API cost modeling, model selection, and ROI calculation for production automation workflows.

---

## Overview

One of the most important skills in AI workflow development is understanding what a system actually costs to run — and being able to justify that cost against the value it delivers. This document analyzes two production workflows from my portfolio using real token measurements and current Anthropic API pricing.

**Key principle:** Every AI workflow has two cost drivers — input tokens (what the model reads) and output tokens (what the model writes). Output is always more expensive. In multi-agent and conversational systems, those costs compound with every message.

---

## Current Pricing Reference

*Source: Anthropic API pricing, March 2026*

| Model | Input (per 1M tokens) | Output (per 1M tokens) | Best for |
|---|---|---|---|
| Claude Haiku 4.5 | $1.00 | $5.00 | Routing logic, short responses, classification |
| Claude Sonnet 4.6 | $3.00 | $15.00 | Complex reasoning, multi-step judgment |
| Claude Opus 4.6 | $5.00 | $25.00 | Highest-complexity tasks only |

**Cost formula:**
```
Cost per run = (input tokens ÷ 1,000,000 × input price)
             + (output tokens ÷ 1,000,000 × output price)

Monthly cost = cost per run × estimated volume
```

---

## Case Study 1 — Arnold Towing Inquiry Responder

### Workflow Overview

A one-shot automation triggered by a Tally form submission. The AI agent reads the inquiry, applies business hours routing logic, and generates two email responses — one to the business owner and one to the customer.

**Architecture:** Webhook → AI Agent → Gmail (×2)
**Type:** One-shot responder (single execution per inquiry, no conversation history)

### Token Measurement

*All values measured directly from the workflow and execution history.*

| Component | Tokens | Type |
|---|---|---|
| System prompt | 177 | Input — loads on every execution |
| Tally form submission | 58 | Input — variable user data |
| Owner notification email | 186 | Output |
| Customer response email | 161 | Output |
| **Total input per run** | **235** | |
| **Total output per run** | **347** | |

### Cost Analysis

| Model | Input cost | Output cost | Cost per run |
|---|---|---|---|
| Haiku 4.5 | $0.000235 | $0.001735 | **$0.00197** |
| Sonnet 4.6 | $0.000705 | $0.005205 | **$0.00591** |

**Model originally configured:** Sonnet 4.6
**Model after optimization:** Haiku 4.5

**Rationale for switch:** This workflow performs three tasks — read structured form data, apply conditional routing logic, write a professional short email. None of these require the complex reasoning that justifies Sonnet's higher cost. Output quality was validated after the switch with no meaningful degradation.

### Monthly Cost Projection

*Estimated volume: 100 inquiries/month (realistic for a small regional towing company)*

| Model | Cost per run | Monthly (100 runs) |
|---|---|---|
| Sonnet 4.6 (original) | $0.00591 | $0.59 |
| Haiku 4.5 (optimized) | $0.00197 | **$0.20** |
| **Monthly savings** | | **$0.39** |

### ROI Calculation

| Metric | Value |
|---|---|
| Time saved per inquiry (manual email writing) | ~10 minutes |
| Monthly inquiries automated | 100 |
| Total time saved per month | ~16.7 hours |
| Owner's estimated hourly value | $50/hr |
| **Monthly value delivered** | **$833** |
| **Monthly API cost (Haiku)** | **$0.20** |
| **Return on investment** | **4,165:1** |

---

## Case Study 2 — Sharp & Clean Barbershop AI Communication System

### Workflow Overview

A conversational multi-agent system serving as a website chatbot for a single-location barbershop. Handles customer inquiries end-to-end — answering FAQ questions, escalating complaints to the owner, and processing appointment bookings — all through a single chat interface.

**Architecture:** Chat Trigger → Orchestrator Agent (Simple Memory) → FAQ Sub-Workflow / Escalation Sub-Workflow / Booking Sub-Workflow
**Type:** Conversational multi-agent (session-based, conversation history persists across turns)

### Architecture Note — Why This Costs More

In a one-shot workflow, the system prompt loads once per execution. In a conversational system with Simple Memory, the system prompt reloads on **every customer message**. In addition, the full conversation history accumulates and is included in every input call. A 6-message session means the orchestrator's system prompt is charged 6 times.

This is the fundamental cost difference between one-shot automations and conversational agents — and it informs model selection decisions.

### Token Measurement

*Measured values noted. Estimated values marked with ~.*

| Component | Tokens | Type | Notes |
|---|---|---|---|
| Orchestrator system prompt | 807 × 6 = 4,842 | Input | Measured — reloads each message |
| User messages | ~240 | Input | Estimated (~40 tokens/message × 6) |
| Conversation history | ~1,200 | Input | Estimated — accumulates each turn |
| Tool definitions (3 sub-workflows) | ~900 | Input | Estimated |
| FAQ system prompt | 267 | Input | Measured — charged per FAQ call |
| Booking/escalation input context | ~200 | Input | Estimated |
| **Total input per session** | **~7,649** | | |
| Orchestrator responses | ~480 | Output | Estimated (~80 tokens × 6 turns) |
| FAQ response | 77 | Output | Measured |
| Booking or escalation response | ~203 | Output | Measured average |
| **Total output per session** | **~760** | | |

*Session model: 6 customer messages, 1 FAQ call, 1 booking or escalation call*

### Cost Analysis

| Model | Input cost | Output cost | Cost per session |
|---|---|---|---|
| Haiku 4.5 | $0.00765 | $0.00380 | **$0.011** |
| Sonnet 4.6 | $0.02295 | $0.01140 | **$0.034** |

**Model configured:** Sonnet 4.6

**Rationale for keeping Sonnet:** Unlike Arnold Towing, this system handles multi-turn conversations where the orchestrator must maintain context across topics, decide which tool to call (or whether to call one at all), collect customer information before escalating, and handle out-of-scope inputs gracefully. These judgment-heavy decisions justify Sonnet's higher cost. A lower-cost model risks degraded routing decisions and poor conversational handling — both visible to customers.

### Monthly Cost Projection

*Estimated volume: 160 sessions/month (~40/week — consistent with client's stated message volume of 30–50/week)*

| Model | Cost per session | Monthly (160 sessions) |
|---|---|---|
| Haiku 4.5 | $0.011 | $1.82 |
| Sonnet 4.6 (current) | $0.034 | **$5.44** |

### ROI Calculation

| Metric | Value |
|---|---|
| Messages handled per month (automated) | 160 |
| Time saved per message (owner responding manually) | ~10 minutes |
| Total time saved per month | ~26.7 hours |
| Owner's estimated hourly value | $50/hr |
| **Monthly value delivered** | **$1,333** |
| **Monthly API cost (Sonnet)** | **$5.44** |
| **Return on investment** | **245:1** |

---

## Comparative Analysis

| | Arnold Towing | Sharp & Clean |
|---|---|---|
| Architecture | One-shot responder | Conversational multi-agent |
| Model | Haiku 4.5 | Sonnet 4.6 |
| Input tokens per run/session | 235 | ~7,649 |
| Output tokens per run/session | 347 | ~760 |
| Cost per run/session | $0.002 | $0.034 |
| Monthly cost | $0.20 | $5.44 |
| Monthly value delivered | $833 | $1,333 |
| ROI | 4,165:1 | 245:1 |

### Key Takeaways

**1. Architecture drives cost, not just model choice.**
Sharp & Clean costs 17× more per session than Arnold Towing — not because of the model, but because conversational systems with memory compound input costs across every turn. Understanding this informs both system design and client pricing.

**2. Model selection should match task complexity.**
Defaulting to the most capable model is not the right approach. Arnold Towing was originally running on Sonnet — a switch to Haiku reduced cost by 67% with no meaningful output degradation. Sharp & Clean warrants Sonnet because the orchestrator makes judgment-heavy decisions across a multi-turn conversation.

**3. ROI justifies cost at any reasonable volume.**
Even at Sonnet pricing, Sharp & Clean delivers $245 in value for every $1 spent. The business case for automation is strong across both workflow types. Cost optimization is about discipline and professional practice — not survival.

**4. Estimated components matter.**
Conversation history, tool definitions, and accumulated context are real token costs that don't appear in the system prompt. In production systems, these should be measured directly from execution logs, not estimated. This analysis uses estimates where direct measurement wasn't available — noted explicitly throughout.

---

*Pricing verified March 2026 via Anthropic API documentation. Token counts measured using Anthropic-compatible tokenizer against live workflow prompts and execution outputs.*
