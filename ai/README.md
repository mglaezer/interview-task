# AI-Assisted Technical Interview Tasks

This guide is designed for technical interviews where the candidate is encouraged to use AI (Cursor, Copilot, etc.). The goal is to see if the candidate can **reason about** the AI's output — not just accept it.

## For Candidates

This is **not** a traditional algorithm interview. You are expected to use AI tools (Cursor, Copilot, Claude, ChatGPT, etc.) throughout the task. Writing code by hand is not the point.

What we're evaluating:
- **How you direct the AI** — the prompts you give, the patterns you ask for, the constraints you set.
- **Your design decisions** — can you recognize when the AI produces a naive solution and steer it toward a cleaner architecture?
- **Your ability to anticipate problems** — edge cases, scalability, concurrency — before they blow up.

Think of yourself as the architect and the AI as a fast but opinionated junior developer. Your job is to lead.

## For Interviewers

### Preparation

Each task requires **starter code** provided to the candidate in advance. This is intentional — we don't evaluate candidates on boilerplate. Prepare the starter code before the interview.

### Design Principles

These tasks are built around failures that AI tools cannot prevent:

- **Bugs in data, not code.** The AI writes correct logic, but the bug is in a config file, test case, or interaction the AI never saw.
- **Contradictory requirements.** Two stakeholders want different things. No amount of code solves a requirements conflict.
- **Interaction bugs.** Two features work correctly in isolation but break at the boundary. The AI implements each feature independently.
- **Trade-offs with no right answer.** The candidate must choose and justify — the AI can list options but cannot make the decision.

### During the Interview

- **Don't ask leading questions.** Give contradictory test cases and let the candidate discover the problem.
- **Use "explain your AI's code" checkpoints.** After the AI generates a fix, ask the candidate to trace through specific scenarios line by line. This is the highest-signal moment in the interview.
- **Watch for the "stop coding" moment.** The strongest signal is when a candidate recognizes that a problem cannot be solved with code and starts asking questions instead.
- **Phases 3-4 are stretch goals.** Each task is designed so that a strong Phase 2 performance is sufficient. Later phases differentiate senior candidates but are not required. Don't rush through Phase 2 to reach Phase 3.

## Tasks

1. [The Silent Assumptions (Billing Engine)](task-1-strategy-trap.md) — AI silently resolves ambiguous business rules
2. [The Race Condition Bank (Concurrent Transfers)](task-2-race-condition-bank.md) — correct code that breaks at runtime under concurrency
3. [The Command Trap (Undo/Redo System)](task-3-command-trap.md) — features that work in isolation but break at the boundary
4. [The Dispatcher Trap (Notification Router)](task-4-dispatcher-trap.md) — cascading failures from data, not code

## Evaluation Criteria

| Behavior | Weak Signal | Strong Signal |
|---|---|---|
| **Diagnosis** | Pastes error into AI, accepts explanation | Traces the bug step-by-step before asking AI |
| **Ambiguity** | Accepts AI's silent assumption | Stops and says "this spec is contradictory" |
| **Trade-offs** | "The AI suggested X" | "X because [constraint], but we lose [trade-off]" |
| **AI output** | Accepts generated code without reading it | Walks through the code and questions specific decisions |
| **Testing** | Tests the happy path | Tests the interaction between features |
| **Production** | "It works on my machine" | "What happens when this crashes / scales / gets bad input?" |
