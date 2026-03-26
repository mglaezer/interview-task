# AI-Assisted Technical Interview Tasks

This guide is designed for technical interviews where the candidate is encouraged to use AI (Cursor, Copilot, etc.). The goal is to see if the candidate can **direct** the AI's architectural choices rather than just accepting its first (usually "naive") output.

## Interviewer's Philosophy

In 2026, we don't test if a candidate can write a loop. We test if they can:

- **Anticipate Complexity**: Spot where the AI is leading them into a "Big Ball of Mud."
- **Enforce Patterns**: Direct the AI to use specific design patterns (Strategy, Command, etc.).
- **Validate Integrity**: Identify where the AI ignores edge cases like concurrency or memory limits.

## Tasks

1. [The Strategy Trap (Billing Engine)](task-1-strategy-trap.md)
2. [The Race Condition Bank (Concurrent Transfers)](task-2-race-condition-bank.md)
3. [The Command Trap (Undo/Redo System)](task-3-command-trap.md)
4. [The Dispatcher Trap (Notification Router)](task-4-dispatcher-trap.md)

## Evaluation Criteria

| Behavior | Junior / "AI-Reliant" Candidate | Senior / "AI-Architect" Candidate |
|---|---|---|
| **Prompting** | "Write code for X." | "Design X using the Strategy pattern." |
| **Logic** | Accepts the AI's if/else logic. | Refactors for the Open-Closed Principle. |
| **Failure** | Surprised when the balance corrupts. | Mentions Race Conditions before coding. |
| **Refactoring** | Asks AI: "Fix this bug." | Asks AI: "Extract this to a separate Service." |
| **Testing** | Tests only the "Happy Path." | Tests for Edge Cases (Memory, Concurrency). |
