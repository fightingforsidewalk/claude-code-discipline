---
name: coding-agent-discipline
description: How a planning chat works with a coding agent (Claude Code or similar) so the human never has to evaluate code — writing a task as a "box" with a key-or-no-key line, scope as actions, docs by filename, observable acceptance criteria and hard stops; installing the CLAUDE.md standing behaviours and handback contract in a repo; and reading the handback that comes back as a plain-English checklist. Use it when writing a task for a coding agent, when a session's output comes back for review, or when setting up a new repo's CLAUDE.md.
---

# Working with a coding agent

The loop: the planning chat writes a **box**; the human pastes it into the agent; the agent
ends with a **handback**; the planning chat reads the handback and writes the next box. The
human decides and carries pastes. The human never evaluates code.

This skill is the planning chat's side. The agent's side is installed in the repo's
`CLAUDE.md` — see `CLAUDE.md.template` at the root of this package, and `ADOPT.md` for the
prompt that installs it.

## Writing a box

Every box has: **Key** (a tracker ID, or *no key* with the reason) · **Goal** (what is true
when done) · **Context** (facts the agent cannot find in the repo — three lines is long) ·
**Scope IN / Scope OUT** (out-of-scope stated as forbidden *actions*, never as another role's
name) · **Steps** with any **STOP** written as a step · **Docs** by filename or *none* with
reason · **Acceptance** as observables, including the behavioural check or *none — tests/docs
only*, and always "handback in the standard shape."

Rules that decide whether a box is good:

- **Gate by risk.** Core logic, migrations, security, money → design handback before code.
  Everything else → build and report.
- **A stop is a line in Steps**, and it survives "let it rip."
- **Never name a gate that does not exist** in the repo. A criterion that cannot fail is not
  a criterion.
- **Never mint an identifier.** A box states the human's key or says *no key*. An agent that
  receives a box with neither will invent one, and a handback header is not an identifier.
- **One box, one coherent commit.**
- **A routine deploy the human performs every time is never a step** or a next action.

Full template and example: `BOX.md`.

## Reading a handback

Three questions, in order: did every acceptance criterion close on an observable · what does
each Surprise get (momentary fix / note on an existing item / — rarely, and only by the human —
a new item) · what did the handback claim that is not yet verified. **A Docs line is a claim
to verify on disk, never a receipt.** A "pushed" is checked against the remote. A layer claim
("verified at the handler") is exactly that layer and no stronger.

Then one to three lines back: acknowledged, recorded where, next box or nothing owed. Any ask
of the human is the last thing in the reply, numbered, under **Needs you**.

Full guidance: `FEEDBACK-LOOP.md`.

## Installing the agent's side in a new repo

Paste `ADOPT.md`'s prompt into a Claude Code session opened *in* the target repo. It reads
the stack, fills the template's placeholders from what it finds, shows the diff, waits for
the go, commits, and proves the contract with one handback about the change itself.

## The rules underneath, cited by name

- *A gate is worth exactly what its negative case can see.*
- *A gate that fails softly is worse than one that does not exist.*
- *An instrument's refusal is not a measurement.*
- *Know what the repository publishes.*
- *Prohibitions are actions, not roles.*
- *A record standing in for a change nobody made* — the changelog that is not the edit, the
  Docs line that is not the receipt.
- *Flag, don't silently decide.*
- *Prove at the layer that enforces.*
- *The ask is the last thing trimmed.*
