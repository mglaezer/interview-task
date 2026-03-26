# Task 3: The Command Trap (Undo/Redo System)

## Goal

A CLI "Virtual Canvas" where users can manage a list of geometric points persisted to a `canvas.json` file.

## Phase 1: The Basics

Ask the candidate to build:

- `add-point x y` — adds a point, returns its ID
- `remove-point id` — removes a point
- `move-point id dx dy` — moves a point by a delta
- `list` — shows all points

The AI produces clean CRUD. Let it.

## Phase 2: Undo

> "Add an `undo` command that reverts the last action."

The AI will implement this — likely with the command pattern or state snapshots. Either is fine at this scale. Don't critique the approach yet.

Have them test it:

```
add-point 0 0      → point 1
add-point 5 5      → point 2
move-point 1 10 10 → point 1 is now at (10, 10)
undo                → point 1 is back at (0, 0)
undo                → point 2 is removed
```

Confirm it works.

## Phase 3: The Compound Operation

> "Add `move-all dx dy` — moves every point by the given delta."

After they implement it, ask:

> "I have 100 points. I run `move-all 5 5`. Now I run `undo`. What happens?"

**What happens:** If the AI implemented `move-all` as a loop of individual `move-point` commands, then `undo` only reverts the last point's move. The user has to hit undo 100 times to fully revert one operation.

**What to look for:**
- Does the candidate notice the problem before or after testing?
- Do they implement **composite commands** — a single undo entry that wraps all 100 moves?
- Or do they restructure so `move-all` is a first-class operation with its own undo logic?

This is a UX bug, not a crash. The code is technically "correct." The AI won't flag it.

## Phase 4: The Branching Problem

Have them add `redo`, then run this sequence:

```
add-point 0 0      → point 1
add-point 5 5      → point 2
undo                → point 2 removed
add-point 3 3      → point 3
redo                → ???
```

Ask: *"What should `redo` do here?"*

**What happens:** The candidate must decide between two models:

| Model | Behavior | Trade-off |
|---|---|---|
| **Linear** (most editors) | Redo stack is cleared when a new action is performed. `redo` does nothing. | Simple, predictable. The "undone future" is lost forever. |
| **Tree** (Emacs, some IDEs) | History branches. `redo` could navigate between branches. | Powerful but complex. Hard to build, hard to explain to users. |

**What to look for:** The AI will almost certainly implement linear history. That's fine — but can the candidate explain *why* that's the right choice for this use case? Can they articulate what the user loses?

There's no right answer. "Linear, because it's what users expect from Ctrl+Z" is a valid justification.

## Phase 5: Memory at Scale (If Time Permits)

> "The canvas now has 10,000 points. The user performs 10,000 `move-all` operations. Profile memory usage."

If the AI used **state snapshots** (memento pattern), this stores 10,000 copies of 10,000 points — ~100M point records. Memory explodes.

If it used **command deltas**, it stores 10,000 small command objects. Memory is flat.

Ask: *"How would you detect this problem before it hits production?"*

**What to look for:**
- Do they add a **history size limit** with a strategy for what happens when it's exceeded (drop oldest entries, compress, persist to disk)?
- Do they mention **benchmarking** or **profiling** rather than just reasoning about it?
- Can they articulate: "snapshots are O(n) per operation, deltas are O(1)" — even approximately?

## Why This Task Works

- Phase 2 lets the AI succeed — the candidate builds momentum with working undo.
- Phase 3 is a **UX bug in correct code** — the AI produces something that works but frustrates users. This is subtle.
- Phase 4 has **no right answer** — it tests whether the candidate can make a deliberate design choice and justify it.
- Phase 5 is an **observable performance failure** — you can measure it, not just argue about it.
- The progression tests: correctness → usability → design judgment → scalability.
