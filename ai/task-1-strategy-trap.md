# Task 1: The Strategy Trap (Billing Engine)

## Goal

Build a CLI tool to calculate shipping costs based on varying business rules.

## Steps

**Step 1.** Ask the AI to create a function that calculates price based on weight (`Weight * 10`).

**Step 2.** Add a requirement for "Distance" (`Distance * 5`) and "Fragile Cargo" (flat +$50).

## The Pivot (The Trap)

Ask the candidate to add a "Holiday Discount" that only applies if the total price is over $200, and a "Regional Tax" that varies by country code.

## The Catch

A "naive" candidate will let the AI build a massive `if-else` or `switch` block inside a single `calculate()` function. This creates **Tight Coupling**.

## What to Look For

Does the candidate instruct the AI to use the **Strategy Pattern**? Can they decouple the "Rules" from the "Engine" so that adding a 10th rule doesn't require touching the existing code?
