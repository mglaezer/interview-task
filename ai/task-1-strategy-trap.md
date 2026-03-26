# Task 1: The Silent Assumptions (Billing Engine)

## Goal

Build a CLI tool to calculate shipping costs based on varying business rules.

## Phase 1: The Basics

Ask the candidate to build:

- `calculate --weight 10 --distance 100`
- Price = `Weight * $10` + `Distance * $5`
- Add a `--fragile` flag that adds a flat +$50 surcharge.

The AI produces a clean function. Let it.

## Phase 2: The Ambiguity

Add two new rules in a single request:

> "Add a Holiday Discount of 10% — but only if the total is over $200. Also add a Regional Tax that varies by country code (US: 8%, DE: 19%, JP: 10%)."

Have them implement it and calculate a shipment: `--weight 15 --distance 200 --fragile --country US --holiday`

Then ask:

> "Is the discount applied before or after tax? Does the tax apply to the discounted price or the original?"

**What happens:** The AI will silently pick an order — probably discount first, then tax. It won't mention the ambiguity. The candidate gets working code that may implement the wrong business logic.

**What to look for:** Does the candidate notice the AI made an assumption? Do they ask *you* (the interviewer acting as product owner) to clarify, rather than just accepting whatever the AI chose?

## Phase 3: Conflicting Rules

Add more rules:

> "Loyalty customers get 15% off. Holiday discount is 10% off. Both apply to the same order."

Ask: *"What's the final price?"*

This has at least three interpretations:

| Interpretation | Formula (on $300 base) | Result |
|---|---|---|
| Additive (25% off) | $300 × 0.75 | $225.00 |
| Compounding (15%, then 10%) | $300 × 0.85 × 0.90 | $229.50 |
| Best discount wins (15%) | $300 × 0.85 | $255.00 |

**What happens:** The AI picks one silently — usually additive or compounding. The candidate gets a number, but can they explain *why* that number?

**What to look for:** Does the candidate recognize there are multiple valid interpretations? Do they define an explicit rule resolution policy rather than accepting the AI's implicit choice?

## Phase 4: Explain the Bill

> "A customer says their shipment cost $347.82 but they expected around $300. Trace how the price was calculated."

Ask the candidate to make the engine produce an itemized breakdown — not just the final number.

**What happens:** The AI's clean `calculate()` function returns a single number. Adding a breakdown requires restructuring — each rule must output its contribution, not just mutate a running total.

**What to look for:**
- Do they add a **pipeline** where each rule returns its adjustment with a label?
- Or do they bolt on logging as an afterthought?
- Can they show: `Base: $250 + Fragile: $50 + Tax (8%): $24 - Holiday (-10%): -$30 = $294`?

A strong candidate recognizes this as an **auditability** requirement, not just a debugging exercise. Billing systems that can't explain their output are a liability.

## Phase 5: The 10th Rule (If Time Permits)

> "We're adding 5 more rules next quarter: bulk weight tiers, insurance, fuel surcharge, weekend delivery premium, and promotional codes. How confident are you that adding them won't break existing rules?"

Don't ask them to implement all five. Ask them to **evaluate their current design** against this future.

**What to look for:**
- Is each rule isolated — can you add one without reading every other rule's code?
- Is there a test that verifies rule X doesn't change the output when rule Y is added?
- Do they mention **regression testing** or **golden file tests** (known inputs → expected outputs)?

A weak answer: "the AI will handle it." A strong answer: "each rule is independent, and I'd add a test fixture with known scenarios that must remain stable."

## Why This Task Works

- The AI produces **correct-looking code at every phase** — the bugs are in silent assumptions, not crashes.
- Phase 2–3 test **ambiguity detection** — a skill AI tools actively undermine by always giving you *an* answer.
- Phase 4 tests **production thinking** — billing systems must explain themselves.
- No design pattern saves you here — the Strategy pattern handles structure, but not rule ordering, conflict resolution, or auditability.
