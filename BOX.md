# Writing a box — the task you hand a coding agent

A **box** is one self-contained task for a coding agent: everything it needs to do the work,
the boundaries it must not cross, and the exact shape of what comes back. The planning chat
writes it; the human pastes it into the agent; the agent ends with a handback (see
`CLAUDE.md.template`). A box is one copy-pasteable block. Nothing is implied — if it matters,
it is written.

## The shape

```
=== BOX: <short title> ===
Key:        <tracker ID> | no key — <why: momentary fix / note on <ID> / not deferred>
Goal:       <one or two lines — what is true when this is done>
Context:    <the two or three facts the agent needs and would not find by reading the repo:
             the decision behind this, the ruling that constrains it, the file that owns the fact>
Scope IN:   <files / modules / behaviours this box touches>
Scope OUT:  <what it must not touch, stated as ACTIONS not roles: "write nothing under
             docs/system/" — never "that belongs to the review chat">
Steps:      1. <...>
            2. <...>
            STOP — <show X and wait for go>   ← only where a gate demands it
            3. <...>
Docs:       <filename — the change> | none — <reason>
Acceptance: - <an observable: an endpoint that now rejects what it used to accept; a test
               count; a field that now appears>
            - <the behavioural check for the deploy, or "none — tests/docs only">
            - handback in the standard shape, Surprises numbered
=== END BOX ===
```

## The rules

- **Key or no key, stated.** A box that names no key invites the agent to invent one. "No key"
  is a valid answer and must be written; the reason says which disposition it is.
- **Gate by risk.** Core logic, migrations, security, and money get a design handback *before*
  code — the plan shown and ruled on first. Everything else builds and reports. A box says
  which it is.
- **Stops are written as stops.** "Generate the migration, show the SQL, wait" is a line in
  the Steps, not a hope in the Context. It survives "let it rip."
- **Prohibitions are actions, not roles.** An instruction must be checkable from the seat of
  whoever must obey it. *"Write nothing under `docs/system/`"* the agent can verify against
  the path it is about to write. *"The review chat owns that"* asks the agent to first know
  who it is.
- **Docs impact is enumerated by filename**, or stated as none with the reason. "Update the
  docs" is not an instruction. A box that names a doc the agent must not write says who
  writes it instead — the agent reports the change, a human makes it.
- **Acceptance criteria are observables.** Never "works correctly"; never a gate that does
  not exist in the repo (see the no-phantom-gates line in CLAUDE.md). A criterion that cannot
  fail is not a criterion.
- **One box, one coherent commit.** A box that needs two commits is two boxes.
- **The box does not narrate.** Context is facts the agent cannot find in the repo; it is not
  the story of how the decision was reached. Three lines is a long Context.

## What the box must never say

- a check that does not run ("lint clean" in a repo with no linter)
- a routine deploy as a step or a next action
- "update the docs" without a filename
- a role name in place of a forbidden action
- "use your judgment" where a stop belongs

## Example

```
=== BOX: bump qs per compliance ruling ===
Key:        no key — dependency bump ruled by compliance; joins no work item
Goal:       qs is at or above 6.16.0 everywhere it appears; nothing else moves.
Context:    Two moderate advisories on the dev-only chain. Compliance ruled bump-not-accept.
            The accepted chain under miniflare (sharp 0.35.2) must NOT move — it is an
            accepted risk with an entry in the register, not a fix target.
Scope IN:   package-lock.json at root and any workspace where qs resolves
Scope OUT:  write nothing under docs/ (report the versions; a human writes the register)
Steps:      1. locate every qs node with `npm ls qs` per workspace
            2. `npm update qs` where present; verify carets already allow it (no package.json edit)
            3. full `npm audit` (no scope flag) — the two qs advisories gone; nothing else changed
            4. verify miniflare's sharp byte-identical before/after
            5. build + tests; commit; push; verify hash on origin/main
Docs:       none — register entry is compliance's call and was ruled "no entry"
Acceptance: - `npm audit` shows the two qs advisories absent and the accepted sharp chain present
            - lockfile diff against HEAD shows qs as the only package that moved
            - behavioural check: none — dev-scoped lockfile change, no shipped output changed
            - handback in the standard shape
=== END BOX ===
```
