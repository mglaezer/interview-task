# Task 4: The Dispatcher Trap (Notification Router)

## Goal

Test whether a candidate can reason about cascading failures, duplicate delivery, and system behavior that only breaks with specific data — not specific code.

## Setup (Interviewer Prepares in Advance)

Provide the candidate with:

- A working CLI: `notify --user alice --channel email --message "Hello"`
- Mock providers (SMS, Email, Push) that log to stdout
- A `users.json` with user preferences and contact info
- A `failover.json` config that defines fallback chains: `{ "email": "sms", "sms": "push", "push": null }`
- Providers accept a `--fail-rate N` flag to simulate failure (e.g., `--fail-rate 0.5` = 50% failure)

The routing and basic failover already work. The interview starts at "here's a working system — now break it."

## Phase 1: The Cycle in the Config (~8 min)

Hand the candidate an updated `failover.json`:

```json
{ "email": "sms", "sms": "email", "push": null }
```

> "A new team submitted this failover config. Deploy it and send a notification to alice via email with `--fail-rate 1` (always fails). What happens?"

**What happens:** Email fails → falls back to SMS → SMS fails → falls back to Email → infinite loop. The system hangs or crashes.

**What to look for:**
- Does the candidate spot the cycle by reading the config before running it? (Strong signal.)
- After observing the loop, how do they fix it? Options:
  - **Visited-set per notification:** Simple, prevents cycles. Doesn't help with non-cyclic retry storms.
  - **Global retry limit:** Caps total attempts. Simple but blunt.
  - **Cycle detection on config load:** Rejects invalid configs upfront. Prevents the problem entirely.
- Do they add **config validation** so this can never happen again, or just patch the runtime?

**Why this is better than hoping the AI writes a loop:** The cycle is in the *data*, not the code. The AI wrote correct failover logic — the bug is in a config file the AI never saw. This reliably produces the failure regardless of how good the AI is.

**Checkpoint — "Explain your fix":** If they add a visited-set, ask: "What if the failover chain is email → sms → push → sms? That's not a cycle from email, but sms appears twice. Does your fix handle it?" This tests whether they understand their own solution's boundaries.

## Phase 2: The Retry Storm (~7 min)

> "Set all providers to `--fail-rate 0.3`. Send 100 notifications in a loop. Watch the output."

With 30% failure and aggressive retry, the system floods providers with retry traffic — each failure generates more attempts, which generate more failures.

Ask: *"How many total provider calls were made for 100 notifications? Is that acceptable?"*

**What to look for:**
- Do they implement **exponential backoff** or a **circuit breaker** (stop calling a provider that's been failing)?
- Can they articulate the trade-off: retry more (higher delivery rate, more load on failing provider) vs. retry less (lower delivery rate, less cascading damage)?
- Do they consider that retrying into a failing provider makes the provider's situation worse?

## Phase 3: The Duplicate (~8 min)

Set email to `--fail-rate 0` (never fails) but add a `--delay 3` flag (responds after 3 seconds).

> "Send a notification via email. The system times out after 1 second and fails over to SMS. But the email was actually delivered — it was just slow. Alice gets the notification twice. Fix it."

This is a real distributed systems problem. Have the candidate implement a solution:

| Approach | Upside | Downside |
|---|---|---|
| Idempotency key per message | Provider can deduplicate | Requires provider support (mock it) |
| Cancel-on-first-success | Reduces duplicates | Race between cancel and delivery |
| Dedup window (ignore same message within N seconds) | Simple | Too short = misses, too long = blocks legit retries |
| Accept it, document it | Honest | Users complain |

**What to look for:** There is no clean answer. The candidate should pick one, implement it, and write a test that verifies: send with 3s delay and 1s timeout → only one delivery logged. They must produce a **code artifact**, not just discuss the options.

## Phase 4: The Audit Trail (~5 min if time permits)

> "A user reports they never received a notification sent 2 hours ago. How do you figure out what happened?"

Ask the candidate to add a `delivery-log.json` that records every notification attempt:

```json
{
  "id": "notif-abc-123",
  "user": "alice",
  "channel": "email",
  "status": "timeout",
  "failover_to": "sms",
  "timestamp": "2026-03-25T10:30:00Z"
}
```

Then ask: *"Show me how you'd trace notification notif-abc-123 from initial send to final delivery (or final failure)."*

**What to look for:**
- Do they assign a **correlation ID** that follows a notification through retries and failovers?
- Can they produce a timeline view: `email: timeout → sms: delivered`?
- Do they distinguish "we sent it" from "they received it"?

## Why This Task Works

- **The cycle is in the config, not the code.** The AI wrote correct failover logic — the bug is in data it never saw. This trap fires reliably regardless of AI quality.
- **Phase 2** is observable — the candidate can count the calls and see the storm.
- **Phase 3** requires a code artifact (dedup implementation + test), not just a discussion.
- **Phase 4** turns "add logging" into a concrete task with a specific deliverable (trace a notification's journey).
- The progression mirrors real production incidents: bad config → overload → duplicate delivery → "where did it go?"
