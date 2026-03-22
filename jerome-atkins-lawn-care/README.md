# Jerome Atkins — Lawn Care Request Tracker

**Built:** March 22, 2026
**Platform:** n8n self-hosted (local Docker)
**Challenge:** Dr. Turing Curriculum — Challenge 3 (Stateful Automation)

---

## The Problem

Jerome Atkins runs a one-man lawn care operation in Columbus, GA. He has about 40 regular customers. Every time someone submits a request, Jerome is working from memory — did he do their back yard last time? Is there a gate code? Was there a complaint to follow up on?

No system. Every request starts from zero.

This workflow fixes that. When a customer submits a job request, Jerome gets a smart briefing delivered to his email before he calls them back. New customers are logged immediately. Returning customers get a context-aware summary of their history.

---

## Node Map

```
Webhook (POST)
  → Normalize Phone (Code — strip non-digits)
  → SQLite Node (CREATE TABLE IF NOT EXISTS)
  → SQLite SELECT (lookup by phone) ← error pin → Gmail - ERROR
  → Check Result (Code — normalize output to yes/no flag)
  → New or Returning? (IF)
      [True — Returning]  → AI Agent → SQLite UPDATE → Gmail - Returning
      [False — New]       → SQLite INSERT → AI Agent → Gmail - New
```

---

## How It Works

**Webhook** receives a POST request with three fields: `name`, `phone`, `job_description`.

**Normalize Phone** strips all non-digits from the phone number. This prevents duplicate records from formatting differences — `614-555-0101` and `6145550101` are the same customer.

**SQLite Node** runs `CREATE TABLE IF NOT EXISTS` on first run to initialize the database. Runs on every execution but only creates the table once.

**SQLite SELECT** looks up the normalized phone number in `jerome.db`. Returns the customer record if found, empty array if not.

**Check Result** normalizes the SELECT output into a clean `customerFound: "yes"/"no"` flag and extracts the record for downstream use.

**New or Returning?** routes based on the flag:
- `yes` → returning customer path
- `no` → new customer path

**Returning path:** AI Agent writes a context-aware briefing using prior history from the DB. UPDATE runs *after* the AI Agent — if the agent fails, no dirty write occurs. Gmail delivers the briefing.

**New path:** INSERT logs the customer immediately. AI Agent writes a first-contact briefing. Gmail delivers.

**Error path:** If SQLite SELECT fails (DB unavailable), the error pin routes to Gmail - ERROR with a plain alert so Jerome still knows a request came in.

---

## Demo

### Scenario 1 — New Customer

**Webhook payload:**
```json
{ "name": "Marcus Webb", "phone": "614-555-0101", "job_description": "Full yard cleanup and edging" }
```

**Email received:**
> Subject: Jerome Lawn Tracker — New Customer
>
> Marcus Webb — Full yard cleanup and edging. First contact, no prior history. Call to confirm property size and schedule.

### Scenario 2 — Returning Customer (same phone)

**Webhook payload:**
```json
{ "name": "Marcus Webb", "phone": "614-555-0101", "job_description": "Hedge trimming and leaf blowout" }
```

**Email received:**
> Subject: Jerome Lawn Tracker — Returning Customer
>
> Marcus Webb — Hedge trimming and leaf blowout. Prior jobs: full yard cleanup and edging (Mar 2026). Call to confirm scope and schedule.

---

## Database Schema

```sql
CREATE TABLE IF NOT EXISTS customers (
  phone        TEXT PRIMARY KEY,
  name         TEXT,
  history      TEXT,
  last_updated TEXT
);
```

- **phone** — normalized (digits only), used as primary key
- **history** — text append format: `"Full yard cleanup | Hedge trimming (2026-03-22)"`
- **DB location** — `/home/node/.n8n/jerome.db` inside Docker container, persisted via named volume `n8n_data`

---

## Known Production Gaps

This is a learning/portfolio build. The following gaps would need to be addressed before real deployment:

| Gap | Notes |
|---|---|
| Unauthenticated webhook | No webhook secret. Anyone who knows the URL can trigger the workflow. Production requires a shared secret header. |
| No encryption at rest | SQLite DB is unencrypted. Production would require an encrypted Docker volume. |
| No backup | DB lives in a Docker volume with no backup cron. A container wipe loses all customer history. |
| PII handling | Phone numbers are PII. Production deployment requires a data retention policy and customer disclosure. |
| Gmail output | Gmail used for build simplicity (OAuth already configured). Production deployment would use Telegram for real-time mobile delivery to Jerome's phone. |
| SQL injection | `name` and `job_description` are inserted into SQL strings without sanitization. Phone is safe (digits-only normalization). Production requires parameterized queries. |

---

## Skills Demonstrated

- **Stateful automation** — data that persists between workflow runs via SQLite
- **Community node** — `n8n-nodes-sqlite3` by DangerBlack (installed via Settings → Community Nodes)
- **Phone normalization** — Code node with `.replace(/\D/g, '')` before DB lookup
- **SELECT / INSERT / UPDATE** — full CRUD pattern across two workflow paths
- **DB output injected into AI Agent context** — prior history becomes part of the briefing prompt
- **IF routing** — new vs. returning customer branching
- **AI Agent security** — prompt injection defense, system prompt confidentiality, job description validation
- **Error path** — SQLite SELECT error pin routed to fallback Gmail alert
- **n8n self-hosted** — local Docker stack, named volume persistence, community node installation

---

## Note on System Prompts

AI Agent system prompts are redacted in this file. The workflow architecture, node logic, and connections are fully visible.
