# Task 3: The Command Trap (Undo/Redo System)

## Goal

A CLI "Virtual Canvas" where users can manage a list of geometric points.

## Steps

**Step 1.** Create commands to `add-point x y`, `remove-point id`, and `move-point id dx dy`.

**Step 2.** Implement an `undo` command that reverts the last action.

## The Pivot (The Trap)

Ask the candidate to implement `redo` and ensure the system can handle a history of 1,000 actions without storing massive "State Snapshots" (Mementos) in memory.

## The Catch

The AI will often suggest saving the entire state of the canvas after every move. This is memory-intensive and fails at scale.

## What to Look For

Does the candidate guide the AI toward the **Command Pattern**? Each action should be an object with `execute()` and `undo()` methods. This allows history to be stored as a list of "deltas" rather than full states.
