# Sharp & Clean Barbershop — AI Communication System

## Project Overview
**Client:** Tony Reeves, Atlanta GA (test/portfolio scenario)
**Problem:** Tony answers 30-50 customer messages/week personally from his phone while cutting hair. Missing messages, losing clients.
**Solution:** Fully automated chat system handling FAQs, appointment booking, and escalation — Tony stays present at his chair.
**Status:** COMPLETE. All 13 test cases passed (Mar 10, 2026). Portfolio-ready.

---

## Tony's Business Data (Locked)

- **Hours:** Tue–Fri 9am–7pm, Sat 8am–5pm, closed Sun–Mon
- **Services & Prices:**
  - Haircut — $35
  - Fade — $40
  - Beard Trim — $20
  - Full Service — $55
- **Walk-ins:** Welcome, appointments preferred
- **Booking:** Via chat bot (this system)
- **Tony's Gmail:** tony@sharpandclean.com

---

## Architecture

```
Chat Trigger
  → Orchestrator Agent (Simple Memory)
      Tools:
        → FAQ Sub-Workflow
        → Escalate Sub-Workflow
        → Book Sub-Workflow
```

---

## Component Specs

### Orchestrator Agent
- Owns the conversation start to finish — speaks as Tony's shop
- Greets customer, gathers context before calling any tool
- **Mixed-intent rule:** Answer FAQ first, then check sentiment — offer escalation if dissatisfaction detected
- Calls FAQ tool for informational questions
- Calls Book tool when customer wants an appointment — collects service + date/time + name + email first
- Calls Escalate tool when needed — collects name + contact (phone or email) first
- If customer declines contact info: warns gracefully ("We may not be able to reach you without contact info") — does not hard-close
- Stays on topic, redirects politely for out-of-scope requests
- Uses Simple Memory (session only — resets between conversations)

### FAQ Sub-Workflow
- AI Agent with Tony's business knowledge in system prompt
- No tools, no memory
- Receives question → returns plain text answer to orchestrator
- Covers: services, prices, hours, walk-ins, appointments, parking — anything informational

### Escalate Sub-Workflow
- Receives: conversation context + customer name + contact info + reason flagged
- Sends Gmail to tony@sharpandclean.com
- Subject: `Customer Escalation — Sharp & Clean`
- Body: one-sentence plain-English reason at top + full conversation + customer name + contact info
- Returns confirmation string to orchestrator
- Trigger conditions: complaints, dissatisfaction, special requests, FAQ couldn't answer, customer asks to speak to someone

### Book Sub-Workflow
- Receives: service + requested date/time + customer name + customer email
- Checks Google Calendar ("Sharp & Clean" calendar) for availability
- **If available:** Creates calendar event (service, name, email, time) → returns confirmation to orchestrator → orchestrator confirms in chat + sends Gmail confirmation to customer email
- **If unavailable:** Reads next available slots → returns 2–3 alternatives to orchestrator → orchestrator presents options conversationally
- Full auto-confirm — no manual approval from Tony
- Calendar event format: `[Service] — [Customer Name]` with customer email in description

---

## Booking Backend

- **Platform:** Google Calendar API via HTTP Request nodes (see Technical Deviations below — native n8n node is broken)
- **Calendar name:** Sharp & Clean (created inside Chris's existing Google account)
- **Calendar ID:** `ffd435b5a4db95e1fa08147695a5b9baf2004ae901bbb64237978256fec73a73@group.calendar.google.com`
- **Credential:** Google Calendar OAuth2 API — credential name: `Sharp and clean calendar`
- **Confirmation email:** HTML Gmail to customer — service, date/time (formatted), shop sign-off

---

## Technical Deviations (What Changed During Build)

### n8n Google Calendar Node — Broken in v2.9.4 (Cloud)
The native n8n Google Calendar node has a confirmed bug: it sends `calendarId` as an array instead of a string, causing a 400 Bad Request on both the availability check and the create event operation. This affects node versions 1.3 and likely earlier.

**Fix:** Replaced ALL Google Calendar nodes with HTTP Request nodes calling the Google Calendar API directly. This gives full control over the request body and bypasses the bug entirely.

#### Freebusy Check (HTTP Request)
- Method: `POST`
- URL: `https://www.googleapis.com/calendar/v3/freeBusy`
- Auth: Predefined Credential Type → Google Calendar OAuth2 API → `Sharp and clean calendar`
- Body: `timeMin`, `timeMax` (RFC3339 with timezone via Luxon), `items` array with calendar ID
- Returns: `calendars.[calendarId].busy` — empty array = available, items = taken

#### Create Event (HTTP Request)
- Method: `POST`
- URL: `https://www.googleapis.com/calendar/v3/calendars/[calendarId]/events`
- **IMPORTANT:** The `@` in the calendar ID must be URL-encoded as `%40` in the URL path
- Auth: same credential
- Body: `summary`, `description`, `start` (dateTime + timeZone), `end` (start + 1hr), `attendees`

### IF Node — Hardcoded Calendar ID Path
`Object.values()` does not work in n8n expression context. The IF node condition must use the full hardcoded calendar ID as a key:
```
{{ $json.calendars["ffd435b5a4db95e1...@group.calendar.google.com"].busy.length }}
```
With "Convert types where required" toggle ON and value type set to Expression (not fixed).

### Datetime Handling
The orchestrator sends `requested_datetime` as `2025-07-17T14:00:00` — no timezone. All datetime fields passed to Google Calendar must be converted to RFC3339 with timezone using Luxon:
```
DateTime.fromISO($('When Executed by Orchestrator').item.json.requested_datetime, {zone: 'America/New_York'}).toISO()
```
End time = start + 1 hour: `.plus({hours: 1}).toISO()`

### Gmail Node — HTML Required
Gmail node must be set to **HTML** message type (not plain text) for line breaks to render. Datetime displayed to customer uses:
```
.toFormat('MMMM d, yyyy \'at\' h:mm a')
```
Which outputs: `March 11, 2026 at 1:00 PM ET`

---

## Decisions Log

| Decision | Choice | Reason |
|----------|--------|--------|
| FAQ as sub-workflow vs. orchestrator prompt | Sub-workflow | Learning goal: two agents in one system |
| Chat trigger channel | n8n Chat Trigger (test) / Instagram (real-world) | Test environment simplification |
| Booking backend | Google Calendar API | Free, Tony-friendly, native n8n node |
| Tony approves bookings? | No — full auto-confirm | Minimize Tony's manual work |
| Unavailable slot handling | Smart alternatives (2–3 options) | Better customer experience than dead-end |
| Contact info for escalation | Collect first, warn gracefully if declined | No hard dead-end, no chasing ghost contacts |
| Mixed-intent messages | FAQ first, then sentiment check + escalation offer | Respects customer's original question |
| Booking confirmation | In-chat message + Gmail to customer | Dual-channel trust signal |
| Google Calendar API approach | HTTP Request nodes (not native node) | n8n v2.9.4 Calendar node bug — sends calendarId as array, causes 400 on all operations |
| Gmail message type | HTML (not plain text) | Line breaks don't render in plain text |

---

## Build Order

1. ✅ **Orchestrator shell** — greeting, routing logic, no tools connected yet
2. ✅ **FAQ sub-workflow** — Tony's business knowledge, plain text returns
3. ✅ **Escalate sub-workflow** — Gmail to Tony, confirmation string return
4. ✅ **Book sub-workflow** — Google Calendar read/write, Gmail confirmation to customer
5. ✅ **Full integration testing** — all 13 test cases passed

> Book sub-workflow requires Google Calendar OAuth connected in n8n credentials before starting.

---

## Pre-Build Checklist

- [x] Create "Sharp & Clean" calendar in Google account
- [x] Connect Google Calendar to n8n via OAuth credential
- [x] Confirm Gmail credential is active in n8n (already exists from prior projects)
- [x] Create new workflow in n8n: "Sharp & Clean — Orchestrator"
- [x] Create sub-workflow stubs: "SC — FAQ", "SC — Escalate", "SC — Book"

---

## Test Cases (Pre-Pilot)

**FAQ**
- [x] Customer asks about hours → correct answer returned
- [x] Customer asks about pricing → all four services listed correctly
- [x] Customer asks about walk-ins → correct policy returned

**Escalate**
- [x] Customer expresses complaint → bot collects info and fires escalation email
- [x] Escalation email arrives at tony@sharpandclean.com formatted and readable
- [x] Customer declines contact info → graceful warning, no crash

**Book**
- [x] Customer books available slot → calendar event created + confirmation in chat + confirmation email sent
- [x] Customer requests unavailable slot → 2–3 alternatives offered
- [x] Double-booking attempt → blocked, alternatives offered

**Orchestrator**
- [x] Mixed-intent message → FAQ answered first, escalation offered after
- [x] Multi-topic conversation → both FAQ and booking handled in one session
- [x] Out-of-scope question → polite redirect

---

## Acceptance Milestone

Pilot approval: Chris completes all test cases successfully → Tony reviews test output and approves a one-week live pilot → pilot runs without critical failure.

---

## Deferred (Not In This Project)

- Multi-session memory
- Automated SMS
- Appointment reminders
- Barber-specific booking (which of the 4 barbers)
- Cancellation / rescheduling flow
