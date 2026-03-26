# Task 2: The Race Condition Bank (Concurrent Transfers)

## Goal

A CLI app that manages bank accounts stored in a local `accounts.json` file.

## Phase 1: The Basics

Ask the candidate to build:

- `create-account --name A --balance 1000`
- `transfer --from A --to B --amount 100`
- Data persists in a JSON file after every transaction.

This phase is a warm-up. The AI will produce clean, working code. Let it.

## Phase 2: Break It

Ask the candidate:

> "Simulate 50 simultaneous transfers of $1 from A to B. Both accounts start with $1000. What should the final balances be?"

The expected answer is A=$950, B=$1050. Have them run it. It won't be.

**What happens:** The AI's "Read -> Modify -> Write" cycle causes lost updates when multiple processes read the same balance before any writes land. The candidate should **observe** the corruption, not just be told about it.

**Interviewer note:** If the candidate says "this will have race conditions" *before* running it — that's a strong signal. Let them explain, then ask them to prove it.

## Phase 3: Fix It — But Choose Your Trade-off

Don't say "add file locking." Instead ask:

> "How would you fix this? What are your options?"

There are several valid approaches, each with real trade-offs:

| Approach | Upside | Downside |
|---|---|---|
| File locking (e.g., `flock`) | Simple, no new dependencies | Doesn't scale beyond one machine |
| In-memory queue/mutex | Fast, no I/O overhead | Lost on crash, single-process only |
| SQLite instead of JSON | ACID transactions built-in | Changes the storage layer entirely |
| Transaction log (append-only) | Crash-recoverable, auditable | More complex to implement |

**What to look for:** The candidate should be able to articulate *why* they're picking one over another. "The AI suggested file locking" is not a justification. "File locking is simplest and we only need single-machine — we can upgrade later" is.

## Phase 4: The Double-Spend

After they fix concurrency, present this:

> "Account A has $100. Two transfers happen simultaneously: $80 to B, and $80 to C. Only one should succeed. Make it work."

This is harder than Phase 2 because the lock alone isn't enough — they need **balance validation inside the critical section**. The sequence must be: lock, read, validate, write, unlock. If they validate *before* locking, the same bug returns.

**What to look for:** Can they identify that the check-then-act must be atomic? This is a classic TOCTOU (time-of-check to time-of-use) problem.

## Phase 5: Crash Recovery (If Time Permits)

> "What happens if the process crashes after debiting A but before crediting B? The money vanishes."

Ask them to handle this. Valid approaches:

- **Write-ahead log:** Log the intent before executing. On startup, replay incomplete transactions.
- **Two-phase write:** Write to a temp file, then atomic rename.
- **Compensating transaction:** Detect partial failures and reverse them.

**What to look for:** Does the candidate reason about **durability**, not just correctness? Do they consider what state the file is in after a crash?

## Why This Task Works

- The AI produces code that **looks correct but fails at runtime** — asking "is this good design?" won't surface the bug.
- Each phase adds a new failure mode that requires **reasoning about execution order**, not just code structure.
- There's no single right answer at any phase — the candidate must **choose and justify** trade-offs.
- The progression (observe failure → diagnose → fix → anticipate next failure) mirrors real incident response.
