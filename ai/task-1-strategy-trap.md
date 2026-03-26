# Task 1: The Silent Assumptions (Billing Engine)

## Goal

Expose whether a candidate notices when AI silently resolves ambiguous business rules.

## Setup (Interviewer Prepares in Advance)

Provide the candidate with:

- A working CLI shipping calculator with `calculate --weight N --distance N --fragile`
- Price formula: `Weight * $10 + Distance * $5 + (Fragile ? $50 : 0)`
- A `test-cases.json` file with 3 passing cases that verify the base logic

This eliminates 5 minutes of boilerplate. The interview starts immediately at the interesting part.

## Phase 1: Add Rules, Get a Bug You Can't See (~5 min)

Give the candidate this spec in one shot:

> "Add two new rules: a Holiday Discount of 10% (only if total exceeds $200) and a Regional Tax by country code (US: 8%, DE: 19%, JP: 10%)."

Let them implement it with AI. It will produce clean, working code.

Then hand them this test case:

```
Input:  --weight 15 --distance 200 --fragile --country US --holiday
Finance team says the correct answer is: $302.40
```

Ask: *"Does your tool match? If not, why not?"*

**What happens:** The AI silently chose an ordering — probably discount-then-tax or tax-then-discount. The finance team's number assumes a specific ordering. The candidate must **reverse-engineer** which ordering produces $302.40 and compare it to what their code does. This is debugging a business logic mismatch, not answering a trivia question.

**What to look for:** Does the candidate trace the calculation step by step, or do they just re-prompt the AI? Can they identify that the ordering is the issue without being told?

## Phase 2: The Impossible Test Suite (~8 min)

Add another rule:

> "Loyalty customers get 15% off."

Then hand them two test cases from two different stakeholders:

```
Test A (from Sales):   weight 20, distance 100, loyal, holiday → expects $229.50
Test B (from Finance): weight 20, distance 100, loyal, holiday → expects $225.00
```

Ask: *"Make both tests pass."*

**What happens:** Test A assumes compounding discounts (15% then 10%). Test B assumes additive (25% off). Both tests cannot pass simultaneously — the business rules are contradictory. The candidate must recognize this is a **requirements conflict**, not a code bug.

**What to look for:**
- Does the candidate try increasingly complex code to make both pass? (Weak — the AI will happily oblige with hacks.)
- Do they stop and say "these specs contradict each other — we need a decision"? (Strong.)
- Do they ask *you* (the product owner) which interpretation is correct? (Strongest.)

## Phase 3: Explain the Bill (~8 min)

> "A customer disputes a charge of $347.82. Make the engine produce an itemized breakdown showing how the total was calculated."

This requires restructuring — the AI's `calculate()` function returns a single number. Each rule must now output its contribution with a label.

**What to look for:**
- Do they refactor toward a **pipeline** where each rule returns `{ label, amount }`, or bolt on logging as an afterthought?
- Does the breakdown actually trace to the final number, or are there rounding gaps?
- Can they show something like: `Base: $250 | Fragile: +$50 | Holiday: -$30 | Tax (8%): +$21.60 | Total: $291.60`?

**Checkpoint — "Explain your AI's code":** Ask the candidate to walk through the refactored pipeline and explain why the rules execute in the order they do. If they cannot justify the ordering, they did not understand the AI's output.

## Phase 4: The 10th Rule (~5 min)

> "Add a Bulk Weight Tier: orders over 50kg get a 5% discount on the weight component only. Add it without changing any existing test outputs."

This is a small, concrete task that tests whether their design is actually extensible:

- Can they add a rule without reading every other rule's code?
- Do the existing test cases still pass after adding the new rule?
- If a test breaks, can they explain why?

**What to look for:** A strong candidate runs existing tests before and after adding the rule. A weak candidate adds the rule and hopes for the best.

## Why This Task Works

- **Phase 1** forces debugging through a contradictory test case — not a leading question.
- **Phase 2** is literally unsolvable with code — it tests whether the candidate knows when to stop coding and start asking questions.
- **Phase 3** requires a code artifact (itemized breakdown), not just a verbal answer.
- **Phase 4** is a 5-minute hands-on extensibility test with a concrete pass/fail criterion.
- Asking the AI "what are the ambiguities?" helps somewhat, but the contradictory test cases force the candidate to **prove** they understand the issue, not just acknowledge it.
