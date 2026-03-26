# Task 4: The Dispatcher Trap (Notification Router)

## Goal

A CLI service that routes notifications to different providers (SMS, Email, Push), with mock providers that can be configured to fail.

## Phase 1: Basic Routing

Ask the candidate to build:

- `notify --user alice --channel email --message "Hello"`
- Providers are mocks that log to stdout: `[EMAIL] Sent to alice: Hello`
- A `users.json` file stores user preferences and contact info (email, phone, device token).

This is a warm-up. The AI will produce a clean router. Let it.

## Phase 2: Priority Routing

Add priority levels:

- `--priority high` → send to ALL channels the user has configured
- `--priority low` → send to Push only

Still straightforward. The AI handles this fine.

## Phase 3: Failover — The Loop

Add configurable failure to the mocks:

> "The Email provider now fails 50% of the time (returns a 500). If Email fails, retry via SMS — but only if the user has a phone number on file."

Have them implement it, then ask:

> "What happens if SMS also fails? And the failover for SMS is Email?"

**What happens:** The AI will likely write linear failover logic (email → SMS) without considering cycles. When you add SMS → Email failover, it creates an **infinite retry loop**. Even if the AI avoids the literal infinite loop, it will probably retry too aggressively — 50 failures in a second.

**What to look for:**
- Does the candidate identify the loop risk before or after running it?
- Do they implement **retry limits**, **circuit breakers**, or a **visited-providers set** to break the cycle?
- Can they articulate the trade-off: retry more (higher delivery rate) vs. retry less (lower system load, fewer duplicates)?

## Phase 4: The Duplicate

After failover works, present this scenario:

> "Email to alice fails. Failover sends via SMS successfully. But the email provider had a network timeout, not a real failure — it actually delivered the email 30 seconds later. Alice got the notification twice."

Ask: *"How do you prevent this?"*

This is a real production problem with no clean solution. Valid approaches:

| Approach | Upside | Downside |
|---|---|---|
| Idempotency key per notification | Provider can deduplicate | Requires provider support |
| Cancel pending retries on first success | Reduces duplicates | Race condition between cancel and delivery |
| Accept duplicates, deduplicate on client | Simple server-side | Pushes complexity to client |
| Time-bounded dedup window | Catches most duplicates | Window too short = misses, too long = memory |

**What to look for:** There's no perfect answer. The candidate should recognize this is a **distributed systems problem** (at-least-once vs. at-most-once delivery) and pick an approach with eyes open about its limitations.

## Phase 5: Observability (If Time Permits)

> "A user reports they never received a notification. How do you figure out what happened?"

Ask the candidate to add enough structure to debug this. The question is deliberately vague.

**What to look for:**
- Do they add a **delivery log** (notification ID, channel, status, timestamp)?
- Do they assign a **correlation ID** that follows a notification through retries and failovers?
- Do they distinguish between "we sent it" and "they received it" — and acknowledge we often can't know the latter?

A weak answer: "add console.log statements."
A strong answer: structured logging with correlation IDs, and an acknowledgment that delivery confirmation depends on the provider.

## Why This Task Works

- Phase 1–2 let the AI shine — the candidate builds confidence and momentum.
- Phase 3 introduces a **runtime failure** (infinite loop) that the AI is unlikely to prevent.
- Phase 4 has **no correct answer** — it tests trade-off reasoning, not pattern knowledge.
- Phase 5 tests **production thinking** — most candidates (and AI tools) stop at "it works on my machine."
- The progression mirrors a real on-call incident: it works → it fails sometimes → it fails in a way that makes things worse → users complain.
