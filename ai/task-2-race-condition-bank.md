# Task 2: The Race Condition Bank (Concurrent Transfers)

## Goal

A CLI app that manages bank accounts stored in a local `accounts.json` file.

## Steps

**Step 1.** Implement a basic `transfer --from A --to B --amount 100` command.

**Step 2.** Ensure the data persists in a JSON file after every transaction.

## The Pivot (The Trap)

Ask the candidate to simulate 50 simultaneous transfers of $1 between the same two accounts using `Promise.all` or a bash script.

## The Catch

Most AI models will write a simple "Read -> Modify -> Write" cycle. When 50 processes read the file at the same time, they will all see the same balance, leading to **Lost Updates** (Data Corruption).

## What to Look For

Does the candidate realize the AI missed **Concurrency Control**? Do they force the AI to implement **File Locking** (Mutexes) or a **Queue/Transaction Log** to ensure atomicity?
