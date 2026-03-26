# Task 2: The Race Condition Bank (Concurrent Transfers)

## Goal

Test whether a candidate can observe, diagnose, and fix runtime failures that AI-generated code produces — then anticipate the next failure mode.

## Setup (Interviewer Prepares in Advance)

Provide the candidate with:

- A working CLI bank app: `create-account`, `transfer --from A --to B --amount N`
- An `accounts.json` file with two accounts: A ($1000), B ($1000)
- A test script (`stress-test.sh`) that runs 50 concurrent transfers of $1 from A to B

The code uses a naive Read → Modify → Write cycle with no locking. The bug is already present. The interview starts at "run this, it's broken."

## Phase 1: Observe and Diagnose (~5 min)

> "Run `stress-test.sh`. Account A should end with $950, B with $1050. Check the actual result."

The balances will be wrong — lost updates from concurrent writes.

Ask: *"What happened? Why are the numbers wrong?"*

**What to look for:**
- Can the candidate explain the Read → Modify → Write race without prompting?
- Do they draw a timeline of two concurrent processes reading the same stale value?
- Or do they just paste the error into the AI and accept its explanation?

**Checkpoint — "Explain the bug":** Ask them to describe a specific interleaving of two transfers that produces a wrong result. Not the concept — a concrete step-by-step trace. This cannot be faked by prompting the AI.

## Phase 2: Fix It — Choose and Justify (~10 min)

> "Fix the concurrency bug. What are your options?"

Let them discuss trade-offs before coding:

| Approach | Upside | Downside |
|---|---|---|
| File locking (`flock`) | Simple, no dependencies | Single machine only |
| In-memory mutex | Fast | Lost on crash, single-process only |
| SQLite | ACID built-in | Changes storage layer |
| Append-only transaction log | Crash-recoverable, auditable | More complex |

Then have them implement their chosen approach and re-run `stress-test.sh`. The test must now pass.

**What to look for:** The justification matters more than the choice. "File locking because we're single-machine and it's the least invasive change" is strong. "The AI suggested it" is weak.

**Checkpoint — "Walk me through the fix":** After the AI generates the locking code, ask the candidate to explain: Where is the lock acquired? What happens if the process dies while holding the lock? Is the lock advisory or mandatory? If they cannot answer, they did not understand the fix.

## Phase 3: Separate Account Files (~8 min, stretch goal)

> "Accounts are now stored in separate files: `accounts/A.json`, `accounts/B.json`. A transfer must debit one file and credit another. Adapt your solution."

This introduces a fundamentally new concern: **multi-resource locking and deadlock**.

If Transfer 1 locks A then B, and Transfer 2 locks B then A simultaneously, they deadlock.

**What to look for:**
- Does the candidate identify the deadlock risk before or after it happens?
- Do they implement **lock ordering** (always lock alphabetically) or another deadlock prevention strategy?
- Can they explain why single-file locking didn't have this problem?

This is genuinely harder than Phase 2 — the AI will produce locking code, but it may not enforce a consistent lock ordering. The bug is in the interaction between two correct lock operations.

## Phase 4: Crash Recovery (discussion only, if time permits)

> "What if the process crashes after debiting A but before crediting B? The money vanishes. How would you handle this?"

This is a **discussion question**, not a coding exercise. Use it to probe the candidate's systems thinking if Phase 3 wraps up early.

Possible directions:
- **Write-ahead log:** Log the intent before executing. On startup, replay incomplete transactions.
- **Two-phase write:** Debit and credit to temp files, then atomic rename both.
- **Compensating transaction:** Detect partial failures on startup and reverse them.

**What to look for:** Does the candidate reason about **durability**, not just correctness? Do they recognize that "what state is on disk after a crash" is a different question from "does the logic work?"

## Why This Task Works

- **Starter code with a pre-built test** eliminates setup time and makes the failure deterministic and observable.
- **Phase 1** tests diagnosis, not pattern-matching — the candidate must trace a specific interleaving.
- **Phase 2** requires a code artifact (fix + passing test), not just discussion.
- **Phase 3** is genuinely different from Phase 2 — it introduces multi-resource coordination and deadlock, not just "more locking." Use it as a stretch goal for candidates who move quickly through Phase 2.
- **Phase 4** is discussion-only — use it to probe systems thinking without eating into coding time.
- **"Explain your AI's code" checkpoints** directly test whether the candidate understands the fix or just accepted it.
- The AI can suggest locking, but it's unlikely to handle lock ordering correctly across separate files without being explicitly directed.
- **A strong Phase 2 performance (correct fix + clear justification) is sufficient.** Phases 3-4 differentiate senior candidates but are not required.
