# Workflow Testing Checklist

**Author:** Chris Arnold
**Date:** March 2026
**Purpose:** A reusable pre-deployment checklist for every n8n automation workflow. Run this in full before calling any build done.

This checklist covers the **workflow layer** — triggers, routing, data flow, error handling, and infrastructure. For AI agent-specific testing (prompt injection, scope enforcement, fabrication defense), see the **Test Before Shipping** section in [ai-agent-template.md](../../../.claude/ai-agent-template.md).

---

## How to Use This Checklist

1. Copy the checklist into a new file in the workflow's project folder (e.g., `sharp-and-clean-barbershop/test-results.md`)
2. Work through every section that applies to your build
3. Mark each item `[x]` when confirmed passing, `[!]` when a known issue exists, or `[n/a]` when not applicable
4. Do not mark a build complete until all applicable items are checked — no exceptions

---

## Section 1 — Trigger

Tests that the workflow starts correctly under all expected entry conditions.

- [ ] **1.1 — Happy path trigger fires** — Send a valid, well-formed input through the trigger. Confirm the workflow starts and reaches the first node.
- [ ] **1.2 — Trigger receives all expected fields** — Confirm every field the workflow depends on (form data, webhook payload, etc.) is present in the trigger output. Pin the first node and inspect the JSON.
- [ ] **1.3 — Unexpected or extra fields don't break anything** — Send a payload with additional fields the workflow wasn't designed for. Confirm it doesn't crash or produce unexpected behavior.
- [ ] **1.4 — Empty or minimal payload** — Send the most minimal input possible. Confirm the workflow handles it gracefully — either processes it correctly or routes to an error path. It should never silently fail or produce a blank output.
- [ ] **1.5 — Trigger method is correct** — For webhooks: confirm the HTTP method (GET vs POST) matches what the sender uses. A method mismatch is a common silent failure.

---

## Section 2 — Data Flow

Tests that data moves through the workflow correctly between nodes.

- [ ] **2.1 — Each node receives the data it expects** — After each key node, inspect the output JSON. Confirm the fields the next node depends on are present and correctly named.
- [ ] **2.2 — Data transformations are correct** — If a Code node or Set node transforms data (e.g., phone normalization, date formatting, field renaming), confirm the output matches the expected format before it reaches downstream nodes.
- [ ] **2.3 — No silent data drops after AI Agent nodes** — After any AI Agent node, only the agent's output exists downstream. Any upstream data needed later must be accessed using `$('NodeName').first().json`. Confirm all references to upstream data use this pattern correctly.
- [ ] **2.4 — IF node conditions evaluate correctly** — For every IF node, manually test both the true path and the false path. If using string comparisons, use `contains` not `equals` — n8n v2.11.4 silently appends a newline to expression fields, which causes `equals` to always fail.
- [ ] **2.5 — Dynamic expressions resolve correctly** — For any field using `{{ }}` expressions, confirm the expression evaluates to the expected value at runtime. Check for undefined, null, or unexpected types.
- [ ] **2.6 — Multi-path routing covers every case** — For any routing node (IF, Switch, or chained IFs), confirm there is a defined path for every possible input. Confirm no input can fall through without a route.

---

## Section 3 — AI Agent Behavior

Tests that the AI agent produces correct, consistent, and safe outputs within the workflow context.

- [ ] **3.1 — Run full Test Before Shipping checklist** — Complete all 8 test cases from the **Test Before Shipping** section in `ai-agent-template.md`. This covers: normal request, prompt injection, unsupported language, missing field, off-topic request, rephrased off-topic (×3), boundary of scope, and input substitution/fabrication.
- [ ] **3.2 — Agent output format matches what downstream nodes expect** — If a node after the agent parses its output (e.g., an IF node checking the agent's category word, a Code node extracting a JSON field), confirm the agent's output format is exactly right. A single extra space, newline, or quote can break parsing.
- [ ] **3.3 — Agent handles missing or malformed input data** — Send a request where a field the agent's prompt references is empty, null, or malformed. Confirm the agent flags it rather than fabricating or guessing.
- [ ] **3.4 — Multi-run consistency** — Run the same input through the agent three times. Confirm the outputs are consistent in structure, tone, and behavior. Occasional variation in wording is acceptable — variation in routing decisions or output format is not.

---

## Section 4 — External Integrations

Tests that connections to external services (APIs, databases, email, calendar, etc.) work correctly.

- [ ] **4.1 — All credentials are valid** — Confirm every credential used in the workflow is authenticated and not expired. Test each one by running the node in isolation.
- [ ] **4.2 — API calls return expected responses** — For every HTTP Request or API node, confirm the response status is 200 (or the expected success code) and the response body contains the expected fields.
- [ ] **4.3 — API rate limits are understood** — Know the rate limit for every external API used. Confirm the workflow won't exceed it under expected volume.
- [ ] **4.4 — Email delivery confirmed** — For any Gmail node, confirm the email arrives in the target inbox. Check subject line, body formatting, sender name, and that no fields are blank.
- [ ] **4.5 — Database reads return correct data** — For any SQLite, Postgres, or other database read, confirm the query returns the expected record for a known input. Test with both a record that exists and one that doesn't.
- [ ] **4.6 — Database writes persist correctly** — For any INSERT or UPDATE, confirm the change is visible in the database after the workflow runs. Confirm it doesn't create duplicate records on repeated runs unless that's the intended behavior.
- [ ] **4.7 — External service failures are handled** — Simulate a failure in each external service (wrong credentials, bad endpoint, etc.) and confirm the workflow routes to the error path rather than crashing silently.

---

## Section 5 — Error Handling

Tests that the workflow fails safely and visibly when something goes wrong.

- [ ] **5.1 — Error output pins are connected** — For any node with an error output pin (HTTP Request, database nodes, etc.), confirm the error path is connected to something — not left floating. A floating error pin means silent failures.
- [ ] **5.2 — Error Alert System fires** — Connect to the account-wide Error Trigger and confirm an alert email arrives when the workflow fails. Trigger a deliberate failure (bad credentials, invalid endpoint) to test this.
- [ ] **5.3 — Error notifications contain useful information** — Confirm the error alert includes: workflow name, failed node, error message, timestamp, and execution URL. Vague error alerts waste debugging time.
- [ ] **5.4 — Error path produces an appropriate user response** — If the workflow sends a message to a customer or user, confirm the error path also sends something — a fallback message, a "we'll be in touch" reply, or an escalation. Never leave a user hanging silently when something fails.
- [ ] **5.5 — Continue on Fail behavior is intentional** — If any node has "Continue on Fail" enabled, confirm this is a deliberate choice for a non-critical step. This setting should never be used to hide failures in critical nodes.

---

## Section 6 — Edge Cases & Volume

Tests behavior under less common but realistic conditions.

- [ ] **6.1 — Unusually long input** — Send an input significantly longer than a typical submission (e.g., a 500-word message into a field that usually gets 20 words). Confirm the workflow handles it without crashing or truncating in a way that breaks downstream logic.
- [ ] **6.2 — Special characters and encoding** — Send inputs containing special characters: apostrophes, quotation marks, ampersands, Unicode characters, emoji. Confirm no node breaks on parsing.
- [ ] **6.3 — Rapid repeated submissions** — Submit the same input multiple times in quick succession. Confirm the workflow handles each one correctly and doesn't create duplicate records, send duplicate emails, or behave unexpectedly.
- [ ] **6.4 — Expected peak volume** — Estimate the maximum realistic number of executions per hour. Confirm the workflow and its external dependencies can handle that volume without hitting rate limits or timeouts.

---

## Section 7 — Security

Tests that the workflow does not expose sensitive data or create exploitable vulnerabilities.

- [ ] **7.1 — No credentials hardcoded anywhere** — Search every Code node, Set node, and HTTP Request node for any API key, password, or token written directly into the workflow. All credentials must use n8n's Credentials system.
- [ ] **7.2 — No sensitive data logged in execution history** — Run the workflow and inspect the execution log. Confirm no API keys, passwords, PII (phone, email, address, payment info), or internal system details appear in node outputs.
- [ ] **7.3 — Webhook endpoint is not publicly guessable** — If the workflow uses a webhook trigger, confirm the URL includes a random token (n8n generates this by default). A predictable URL is a security risk.
- [ ] **7.4 — SQL queries use parameterized inputs** — For any database node that incorporates user-provided data into a query, confirm the input is parameterized — never concatenated directly into the SQL string. Direct concatenation creates SQL injection vulnerabilities.
- [ ] **7.5 — AI agent security tests passed** — Confirm test cases 2, 4, 6, 7, and 8 from the agent template checklist (prompt injection, off-topic ×3, boundary of scope, input substitution) all produced the expected refusal or validation behavior.

---

## Section 8 — Final Verification

A final check before marking the workflow complete.

- [ ] **8.1 — Full end-to-end run with real data** — Run the workflow once with a complete, realistic input — not a test stub, not a minimal payload. Confirm every node executes, every output looks correct, and the final action (email sent, record written, message delivered) is confirmed.
- [ ] **8.2 — All execution paths tested** — Confirm every branch of every routing node has been tested at least once. No untested code paths.
- [ ] **8.3 — Workflow is named clearly** — The workflow name in n8n describes what it does. Anyone looking at the workflow list should understand its purpose without opening it.
- [ ] **8.4 — System prompt is versioned** — The agent's system prompt includes a version line at the top (`Version: 1.0 — YYYY-MM-DD`) per the agent template standard.
- [ ] **8.5 — GitHub is up to date** — The workflow JSON has been exported, system prompt redacted, and pushed to the repo. The workflow's README or changelog reflects the current state of the build.
- [ ] **8.6 — Error Alert System is active** — The account-wide Error Trigger workflow is published and connected. Confirm it's running before deploying anything new.

---

## Quick Reference — Known n8n Gotchas

These are documented platform bugs and quirks that have caused real failures in production. Check these proactively.

| Issue | Affected versions | Symptom | Fix |
|---|---|---|---|
| Expression editor trailing newline | n8n v2.11.4+ | IF node `equals` comparisons always fail silently | Use `contains` instead of `equals` for all string comparisons |
| Switch node broken with expressions | n8n Cloud (ongoing) | Switch node routes incorrectly when output values use `{{ }}` expressions | Replace Switch node with chained IF nodes |
| Google Calendar node broken | n8n v2.9.4 | Sends `calendarId` as an array, causing API rejection | Use HTTP Request nodes to call Google Calendar API directly |
| AI Agent node drops upstream data | All versions | Fields from nodes before the AI Agent are not accessible after it | Access upstream data with `$('NodeName').first().json` |

---

*Maintained by Chris Arnold. Update this document when new platform bugs or workflow patterns are discovered.*
