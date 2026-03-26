# Task 3: The Command Trap (Undo/Redo System)

## Goal

Test whether a candidate can catch interaction bugs between independently correct features, and make deliberate design trade-offs the AI cannot resolve.

## Setup (Interviewer Prepares in Advance)

Provide the candidate with:

- A working CLI canvas: `add-point x y`, `remove-point id`, `move-point id dx dy`, `list`
- Points persist to `canvas.json`
- 5 points pre-loaded so they don't start with an empty canvas

This eliminates CRUD boilerplate. The interview starts at the design problem.

## Phase 1: Undo (~5 min)

> "Add an `undo` command that reverts the last action."

The AI will implement this — likely with the command pattern or state snapshots. Either approach is fine at this point. Have them verify:

```
add-point 0 0      → point 6
move-point 6 10 10 → point 6 is at (10, 10)
undo                → point 6 is back at (0, 0)
undo                → point 6 is removed
```

Don't critique the approach yet.

## Phase 2: Groups — The Interaction Bug (~10 min)

> "Add `group create id1 id2 ...` (creates a named group) and `group move group-id dx dy` (moves all points in the group)."

Let the AI implement it. Then have them run this sequence:

```
group create g1 1 2 3
group move g1 5 5
undo                     → group move is reverted, points back to original positions
undo                     → group g1 is removed
undo                     → ??? what happens to the next undo?
redo                     → ??? does the group come back?
redo                     → ??? does the group move re-apply? Do the points still exist in the group?
```

**What happens:** The AI will implement groups and undo separately, each correctly in isolation. But the interaction between them creates subtle bugs:

- If undo removes the group, does redo restore it with the same members — even if those points were modified in between?
- If a point is deleted after being added to a group, and then the user undoes the group creation, what happens?
- Does `group move` create one undo entry or N entries (one per point)?

**What to look for:**
- Does the candidate discover these bugs by testing, or does the AI somehow handle all of them? (If the AI handles them, ask the candidate to explain *how* — that's the checkpoint.)
- Do they implement a **composite command** for `group move` so that a single undo reverts all point moves at once?
- Can they articulate the invariant: "undo/redo must maintain referential integrity between groups and points"?

**Checkpoint — "Explain your AI's code":** Point to the group move undo logic. Ask: "If I delete point 2 after this group move, then undo the group move, what happens to point 2's position?" The candidate must trace through their code — the AI cannot answer this about its own generated code without context.

## Phase 3: Redo and the Branching Decision (~7 min)

> "Add `redo`. Then run this sequence:"

```
add-point 0 0      → point 6
add-point 5 5      → point 7
undo                → point 7 removed
add-point 3 3      → point 8
redo                → ???
```

Ask: *"What should redo do here? Why?"*

The AI will implement linear history (clear redo stack on new action). That's probably correct — but the candidate must justify it:

| Model | Behavior | Trade-off |
|---|---|---|
| **Linear** (most editors) | Redo stack cleared on new action. Simple, predictable. | The "undone future" is lost forever. |
| **Tree** (Emacs, some IDEs) | History branches. Redo navigates branches. | Powerful but complex UX. |

**What to look for:** Not which model they choose — but whether they recognize there IS a choice. "That's just how undo works" is weak. "Linear, because the branching UX would confuse users in a CLI tool" is strong.

## Phase 4: Persistent History (~5 min if time permits)

> "The undo history must survive process restarts. When I reopen the tool, I should be able to undo my last session's work."

This forces serialization of commands to disk — which raises real design questions:

- What format? JSON commands? A log file?
- What if the code changes but old undo history references a command type that no longer exists?
- How big can the history file get? Do you cap it?

**What to look for:** This is a constraint that defeats AI pattern-matching. The AI can serialize commands, but handling versioning and forward compatibility requires judgment. Does the candidate think about what happens in 6 months when someone adds a new command type?

## Why This Task Works

- **Starter code** skips 5 minutes of CRUD boilerplate.
- **Phase 2's bug is in the interaction** between two features (groups + undo), not in either feature alone. The AI implements each correctly — the bug emerges at the boundary.
- **Phase 2's test sequence** is concrete — the candidate must run it and observe what happens, not just discuss it.
- **Phase 3** has no right answer — it tests design judgment the AI cannot provide.
- **Phase 4** introduces a constraint (persistence) that creates real decisions about format, versioning, and limits.
- The "explain your AI's code" checkpoint in Phase 2 catches candidates who accepted the AI's output without understanding the interaction.
