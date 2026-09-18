# Ship check

**What the agent runs before code leaves its hands — and why the list is the small part.**

The loop in this package ends with a handback: the agent reports, the planning chat reads it
as plain English, the human decides. That loop rests on one assumption nobody states
out loud — **that the handback's claims are worth reading.** A handback saying "fixed, guard
added, tests pass" is four words of evidence and an unknown quantity of hope.

This document is the other side of that. It is the check the agent runs on its own output
before the handback is written, and it exists so the sentences in the handback are reports
rather than feelings.

The short form ships as a skill at `skills/ship-check/SKILL.md`; this file is the reasoning
behind it. Install the skill, or paste its checklist into the repo's `CLAUDE.md` under standing
behaviours. Either works. What matters is that it runs before the handback, not after the
human asks. The skill is the canonical list; if the two ever disagree, the skill is right and
this file is behind.

---

## Read this before you read the list

**Sections 1 to 4 below are commodity.** Do not build a shell string from input, parameterise
your SQL, allow-list then containment-check a path, do not commit secrets, ship a
`.gitignore`, write atomically. Every security checklist written this century has them and
they are not a contribution — they are here because a checklist with holes in the obvious
places is worse than no checklist, since it reads as complete.

**The part that earns its place is sections 5 and 6**, and specifically three ideas:

- **A check you ran silently is indistinguishable from one you skipped.** The result is stated
  in the reply, or the check did not happen.
- **A gate is worth exactly what its negative case can see.** Feed every guard the input it
  should refuse and *show the refusal*. A guard nobody has watched fail has not been proven to
  exist.
- **When fixing a defect, the order is the control.** Reproduce it first. Verify only after
  the fix and a green result cannot tell you whether the guard works or the hole was never
  there.

If you take nothing else from this document, take those three and throw the rest away.

---

## Why this belongs in a package about working with an agent

Because the failure it prevents is specific to this loop.

When a human writes the fix and a second human reviews it, the review is the check. In this
loop **the same party writes the code, tests it, and reports on it** — three roles, one
author, and the human at the end has explicitly agreed not to read the code. That
arrangement works right up until the report is the only artifact and the report is generous
about itself.

So the discipline has to move into the reporting. Not "be careful" — a list, run against the
final tree, with the result stated and the parts that did not apply named.

Two rules from `CLAUDE.md.template` do half of this already and are worth re-reading in this
light: *never name a gate that does not exist* (a criterion that cannot fail is not a
criterion) and *the handback's docs line is a claim to verify, never a receipt*. This
document is the same principle applied to the agent's own evidence.

---

## The check

**State the result explicitly in the handback**, and never as a bare "clean": what was
fixed, or "clean" followed by which sections did not apply and why. Section 6 says why the
bare word is useless.

### 1. Input that becomes something else

- **Input → filesystem path.** Restrict to `[A-Za-z0-9_-]+` (or the tightest set that works),
  then resolve the final path and assert it stays inside the intended base directory. Two
  layers — the allow-list *and* the containment check — never one. Test with `../../x`,
  `x/../../y`, `..`.
- **Input → shell command.** Never build a shell string from input. Argument lists, never
  `shell=True` with interpolated text. If a shell is unavoidable, quote every piece and say
  why.
- **Input → format string / regex / SQL / HTML.** Format-string evaluation on user text can
  read attributes; a regex built from input can backtrack catastrophically; SQL takes
  parameters, not interpolation; HTML gets escaped. Ask of every input: **what language does
  it land in?**
- **Input → file content that is also syntax.** If a file has structural markers — comment
  delimiters, fences, front-matter — refuse input containing them, or escape it.

### 2. Things that must not be in the files

- **Secrets.** No keys, tokens, passwords, connection strings, private hostnames. Reference
  secrets by location and rotation path, never by value. Grep for `key`, `token`, `secret`,
  `password`, `://`.
- **Names**, when the artefact is for others: people, company, products, internal role names,
  local paths. Include the LICENSE holder line — a neutral holder unless attribution was
  asked for.
- **Unresolved placeholders.** `TODO`, `TBD`, `<your-handle>`, `example.com`. Fill them,
  remove them, or name them explicitly as the human's to fill. Never leave one that looks
  finished.
- **Session-specific paths** from wherever the agent was working.
- **Invisible characters written by accident.** Literal control characters, NUL, U+2028 and
  U+2029 can land in a file when tooling decodes an escape sequence that was typed as text.
  They are invisible in an editor and in a diff, and one careless edit removes them with
  nobody the wiser. Scan the final bytes for anything below `0x20` that is not newline or
  tab, plus `0x7F`, U+2028 and U+2029 — and in source, write them as escape sequences rather
  than literals.

### 3. Repo hygiene

- `.gitignore` present. Do not ignore folders the human's own data will live in.
- `LICENSE` present and its holder intentional.
- Every referenced file exists; README links resolve; a layout section matches the tree's
  visible files — dotfiles and repo housekeeping (`.gitignore`, `.editorconfig`, `.github/`
  and the like) are not listed there; a section number cited in one file exists in the other.
- **Front-matter parses.** A Markdown file with YAML front-matter goes through a parser
  before packaging, never an eyeball: a `description:` value carrying a colon-space is
  quoted, or YAML reads the colon as a nested key and a strict loader refuses the file.
- Scripts have a docstring or `--help` matching what they do **now**, not two edits ago.

### 4. Writes and failure

- **Atomic writes**: temp file then an atomic replace, so a crash mid-write leaves the old
  file intact.
- **Assert before you mutate**: exact-match counts, marker counts, structural invariants —
  checked *before* the write, so a failed assertion leaves the file untouched.
- **Fail loudly**, with a reason a human can act on. Never swallow an exception to keep going.
- **Refuse the dangerous case** rather than warn about it, when refusing costs nothing.

### 5. Prove the guards — and prove them in the right order

**A gate is worth exactly what its negative case can see.** For every refusal written, feed it
the input it should refuse and show the refusal in the handback. Then run the happy path once
more to show the guard did not break it.

**When fixing a defect rather than writing a new guard, the order is the control.** Reproduce
the defect against the unfixed code and show it happening, *then* fix, *then* show it refused.

Verify only after the fix and a green result cannot distinguish **the guard works** from **the
defect was never there**. You have proven that the code does not currently do the thing you
already stopped it doing, which is a different claim. A fix reported with no red step behind
it reads identically whether or not the hole ever existed — and the case where it did not is
worse than it sounds, because now there is a guard nobody understands protecting against
nothing, and a handback that says the problem is solved.

If reproducing it is genuinely impractical — it needs production data, a third party, or a
race that cannot be forced — **say so, and say what was checked instead.** *"Could not
reproduce; the guard refuses the input the advisory describes"* is an honest report. Silence
about the missing red step is not.

**Scope, so this stays a control and not ceremony.** It applies when the claim is *a defect is
fixed* or *a guard works* — security, correctness, anything where a passing result is the
evidence. It does not apply to documentation, copy, formatting, renames, or any change with
no refusal behaviour to demonstrate. Those have nothing to go red, and inventing a step for
them buys nothing. This is the same gate-by-risk principle the box format already uses.

**The same rule governs the comment left behind in the code.** A comment asserting a
verification is itself a claim, usually written by whoever did the fixing about their own
work, so it carries **who established it and how**. A comment is evidence of what someone
believed, never evidence of what is true.

### 6. Before packaging

- Run the greps from section 2 and the tests from sections 1 and 5 **on the final tree, not on
  the draft**. Edits made after the check are unchecked.
- **List every file** going out, so the human can see the shape of it without reading it.
- **Name any section that did not apply, and why.** A docs-only change ships no executable
  code, so sections 1, 4 and 5 have nothing to run — saying that is different from claiming
  clean on checks that never ran. Bare "clean" covers both, which is exactly what makes it
  useless.
- If a second reviewer is available — another chat, a linter, a scanner — say so and invite
  it.

---

## What this looks like in a handback

Bad, and the most common shape:

> Fixed the path handling. Added validation. Tests pass.

Good, taken from a real fix in the sibling repository
[skill-claude-relay](https://github.com/fightingforsidewalk/skill-claude-relay), which is
where `relay.py` and the panes directory live:

> **ship-check: clean.** Four files going out: `relay.py`, `templates/pane.md`, `README.md`,
> `.gitignore`.
>
> The traversal: reproduced first against the unfixed script — `--role ../../tmp/probe` wrote
> `/tmp/probe.md`, outside the panes directory, shown below. Fixed with an allow-list plus a
> realpath containment check. Re-ran the same three inputs (`../../tmp/probe`, `x/../../y`,
> `..`): each refused with `invalid role name`. Happy path re-run: `--role build` still
> writes `panes/build.md`.
>
> Sections 1 to 4 otherwise: no secrets, no names, no placeholders, `.gitignore` present,
> LICENSE present and intentional. Writes are temp-file-then-replace with a
> count assertion before each.
>
> Section 5 has nothing else to run — no other refusals in this change.

The second one is longer and it is the only one a human can act on. It says what was
demonstrated, in what order, and what was not checked.

---

## Where the skill lives

`skills/ship-check/SKILL.md`, beside `skills/coding-agent-discipline/`. It installs the same
way: copy the folder into `.claude/skills/` in the repo, or upload it under
*Settings → Capabilities → Skills*, or paste its checklist into `CLAUDE.md` under standing
behaviours. `ADOPT.md` installs it with the rest.

---

## The general forms

**A check you ran silently is indistinguishable from one you skipped.**

**A gate is worth exactly what its negative case can see.**

**Verification after the fact cannot tell you whether the fix mattered.** The red step is not
ceremony; it is the only thing that makes the green step mean anything.

**A comment is evidence of what someone believed, never evidence of what is true.**

**A bare "clean" covers both *I ran it all* and *most of it did not apply and I did not
mention that*.** Name what did not run.
