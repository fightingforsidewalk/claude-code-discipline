# The feedback loop — what happens on either side of a box

The box goes in; a handback comes out; the planning chat reads it and writes the next box.
The human carries the paste in both directions and makes the decisions. Nothing else is the
human's job — in particular, the human never evaluates code. What follows is how each side
behaves so that stays true.

## When the agent's output comes back

The planning chat reviews it as a **plain-English checklist** of what the change should do,
not as a code review. Three questions, in order:

1. **Did every acceptance criterion close on an observable?** A line that says "done" without
   the number, the hash, or the behaviour is not closed. Ask for the observable.
2. **What did the Surprises section say — and what disposition does each surprise get?**
   Most are a momentary diversion (fixed, nothing written). Some are a note on an existing
   item with a trigger. The rarest is a new tracked item — and only the human mints one.
   A handback whose Surprises say "none" on a non-trivial task is read twice.
3. **What did the handback claim that has not been verified?** A Docs line is a claim to
   verify on disk, never a receipt. A "pushed" is verified against the remote. "Verified at
   the handler layer" is not "verified at the database."

Then: one line of acknowledgement, and either the next box or "nothing owed."

## When something in the handback is wrong

- **A self-caught error reported honestly is a good handback.** Bank it; do not punish it.
- **A deviation without an argument** is sent back for the argument, not the reversal —
  the agent may have been right.
- **A claimed edit that did not land** (a Docs line naming a file that is unchanged on disk)
  is the most important catch in this loop. It is the record standing in for the change.
  Verify every such line; never bank it from the handback text.
- **A phantom identifier** in a handback header — the agent numbered its own work — is not
  an identifier. Match the work to a row, never the label.

## Length

A box is about twenty-five lines. A handback is as long as its Surprises need and no longer.
The planning chat's acknowledgement is one to three lines. The full reasoning behind any of
it lives in one place — the project's docs — and everything else points there. Cutting length
never cuts an ask: if the human must do, decide, or look at something, it is the last thing
in the reply, numbered, under **Needs you**.

## Everything the human is never asked to do

- relay a message between chats (the relay protocol exists for that)
- evaluate a diff
- re-run a check the agent could have run
- decide something the docs already decided
- remember a ruling — the docs win over anyone's memory, including the human's
