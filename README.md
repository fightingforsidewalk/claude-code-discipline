# claude-code-discipline

**A working contract between a planning chat, a coding agent, and a human who never has to
read a diff.**

Four pieces, all plain Markdown:

- **`CLAUDE.md.template`** — what the coding agent reads at the start of every session: scope
  walls, the real commands, the gates that exist (and the ones that don't), standing
  behaviours, and the **handback contract** — a fixed block every session ends with.
- **`BOX.md`** — how the planning chat writes a task for the agent: key or no key, scope as
  actions, docs by filename, acceptance as observables, stops as steps.
- **`FEEDBACK-LOOP.md`** — how the planning chat reads what comes back, and everything the
  human is never asked to do.
- **`SHIP-CHECK.md`** — what the agent runs on its own output before the handback is written,
  so the handback's sentences are reports rather than feelings: reproduce a defect before
  fixing it, show every guard refusing, name what did not apply.

Plus **`ADOPT.md`**, one prompt that installs the agent's side in any repo, and two Claude
**skills**: `skills/coding-agent-discipline/` for the planning chat's side, and
`skills/ship-check/` for the agent's pre-ship check.

Companion repos: **[skill-claude-relay](https://github.com/fightingforsidewalk/skill-claude-relay)**
(the mailbox and operating model for several chats on one project) and
**[skill-canonical-tracker](https://github.com/fightingforsidewalk/skill-canonical-tracker)**
(one record, many surfaces, a check that proves they agree). This one is the third leg: how
code gets written and reported.

---

## The problem

A coding agent that is asked to "fix the thing" will fix the thing and tell you it's fixed.
What it will not tell you, unless the contract demands it, is that it also changed a second
file, that the test count went down by one, that it noticed the deploy pipeline does
something the docs don't say, or that it broke something and quietly repaired it on the way.
All four of those are the information you actually needed.

The contract here came out of running one agent under a planning chat on a real product.
Every line is there because something went wrong without it. The centrepiece is the
**handback** — nine fixed lines, the last of which is **Surprises**, numbered — and the
rule that makes it work: **flag, don't silently decide.**

## What a handback looks like

```
=== HANDBACK → planning chat ===
Task:          Bump qs per compliance ruling — two dev-chain advisories
Status:        done
Files changed: package-lock.json — qs 6.15.3 → 6.16.0. NO package.json changed.
Build+tests:   typecheck exit 0 · build exit 0 · 134 passed 0 failed · core file in diff? NO
Git:           779f42c — pushed, verified on origin/main.
Deploy:        Push fired the frontend auto-deploy; qs is not in the shipped bundle; output unchanged.
Verified:      Audit layer — the two advisories GONE; the accepted chain still present, byte-identical.
               NOT reached: nothing rendered. No behavioural check exists here, and I claim none.
Next:          nothing.
Surprises:     1. qs lives in ONE workspace, not the three the box allowed for.
               2. The accepted-chain alerts were dismissed DURING this box by someone else —
                  "0 open alerts" now means accepted, not remediated.
=== END HANDBACK ===
```

Surprise 2 is the whole reason the format exists. `examples/handback-example.md` has the
full version and what the planning chat did with it.

## The rules that carry the weight

- **Flag, don't silently decide.** Anything the task did not anticipate is numbered under
  Surprises. Silently skipping and silently doing extra are equally wrong.
- **Prove at the layer that enforces.** "The handler rejects it" is not "the database refuses
  it." Name the layer you reached and claim only that one.
- **A behavioural check, or "none" with a reason.** "Build succeeded" is never the check.
- **Never name a gate that does not exist.** A criterion naming a linter in a repo with no
  linter closes on nothing, indistinguishably from passing. And a gate that fails *softly* is
  worse than one that is missing — it prints something reassuring on the way past.
- **Know what the repository publishes.** If the repo root is the document root of a deployed
  site, every file an agent adds is served on the internet by default. Check what a URL returns
  rather than what the config says it should.
- **Never reproduce a secret.** Not in chat, not in a file, not in a handback. Secrets are named
  by where they live, never by value.
- **Stops are hard.** "Show the SQL and wait" survives "let it rip."
- **Self-caught errors are reported, not cleaned away.** The near-miss is information.
- **A Docs line is a claim to verify, never a receipt.** The planning chat checks on disk.
- **Prohibitions are actions, not roles.** *"Write nothing under `docs/system/`"* is checkable
  from the agent's seat; *"that belongs to another chat"* is not.
- **Only the human mints identifiers.** A handback header is not an identifier.
- **The ask is the last thing trimmed.** Every reply that needs the human ends with a
  numbered *Needs you* list.
- **Reproduce before you fix.** A green result after the fix cannot tell "the guard works"
  from "the defect was never there". Show the red step, then the fix, then the refusal, and
  say which sections of the check did not apply rather than reporting a bare "clean".

## Quick start

1. Open a Claude Code session in your repo and paste the prompt from [`ADOPT.md`](ADOPT.md).
   It reads the stack, fills the template, shows you the diff, and proves the contract with
   one handback about itself.
2. Give your planning chat the skill in `skills/coding-agent-discipline/` (or paste its
   `SKILL.md` into the project instructions).
3. Write the first box from [`BOX.md`](BOX.md). Read the first handback with
   [`FEEDBACK-LOOP.md`](FEEDBACK-LOOP.md) open.
4. When a handback says a defect is fixed or a guard works, expect the shape in
   [`SHIP-CHECK.md`](SHIP-CHECK.md): the red step, the refusal, and the sections that did
   not apply.

## Layout

```
README.md
ADOPT.md                      the prompt that installs the agent's side in a repo
CLAUDE.md.template            scope · stack · commands · architecture · database · deploy gates ·
                              sources of truth · standing behaviours · handback contract
BOX.md                        how to write a task for the agent, with an example
FEEDBACK-LOOP.md              how to read what comes back; what the human never does
SHIP-CHECK.md                 the agent's check on its own output before the handback, with the reasoning
examples/handback-example.md  a real handback and its disposition
skills/coding-agent-discipline/SKILL.md   the planning chat's side, as a Claude skill
skills/ship-check/SKILL.md    the agent's pre-ship check, as a Claude skill (the canonical short form)
LICENSE                       CC0 1.0
```

## License

[CC0 1.0](LICENSE) — public domain, no attribution required. Change the line names, keep the
Surprises section, and if an agent ever surprises you in a way this contract did not catch,
that is a new line for the standing behaviours — add it with the date and the incident, the
way every line here was added.
