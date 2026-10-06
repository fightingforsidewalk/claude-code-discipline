# Adopting this in a repo

Paste the prompt below into a **Claude Code session opened in the target repo** — that is the
seat that can read the stack and write `CLAUDE.md`. If the repo already has a `CLAUDE.md`, the
prompt adds to it; nothing is removed. The planning-chat side (`skills/coding-agent-discipline/`)
installs like any skill: copy the folder into `.claude/skills/` or upload it under
*Settings → Capabilities → Skills*; or paste `SKILL.md` into the planning chat's project
instructions. The agent's pre-ship check (`skills/ship-check/`) installs the same way on the
agent's side, and the prompt below copies it into the repo.

## The prompt

```
You are adopting a standing set of behaviours for this repository: rules for every coding
session, and a fixed HANDBACK block that ends every session that changes anything.
Nothing in it is specific to the project it came from except the placeholders in angle
brackets, which you will fill from THIS repo.
The template is the file CLAUDE.md.template in the package I have given you (or pasted below
this prompt). Do the following in order.

1. READ THIS REPO FIRST. Identify: the build command(s); the test command(s) and the current
   passing count; whether a linter is actually configured (config file AND dependency AND
   script — if any is missing, there is no linter and no criterion may ever name one); the
   deploy targets and what a push to the main branch sets in motion for each; the database
   and how migrations are generated and applied; any files that count as "core" (an engine,
   a schema, a payments path) where a change deserves to be called out separately; and any
   sibling repos on this machine that must be named as off-limits.

2. INSTALL THE SHIP CHECK. Copy skills/ship-check/ from the package into .claude/skills/ in
   this repo, so it loads in every session here. If this repo already has a ship-check skill,
   show me the diff between the two and wait; do not overwrite it.

3. WRITE CLAUDE.md at the repo root from CLAUDE.md.template (create it if absent; if it
   exists, add the sections it lacks and merge the ones it has, removing nothing). Replace
   every <placeholder> with what you found in step 1. Where a placeholder does not apply,
   delete the line rather than leave it vague — a rule that names a check that does not
   exist is worse than no rule. Keep the Standing behaviors and Handback contract sections
   verbatim except for the placeholders.

4. SHOW ME THE DIFF to CLAUDE.md before committing it. List every placeholder you filled and
   what you filled it with, and every line you deleted as not-applicable. Wait for my approval.
   Do not commit, push, or run anything else before I approve.

5. AFTER I APPROVE: commit it (message in this repo's existing style — read the last ten commit
   messages first), push, verify the hash on the remote — and end this session with a
   handback in exactly the shape the new CLAUDE.md specifies, describing this change. That
   handback is the proof the contract is installed; if it is missing a line, the installation
   is not done.

6. END WITH "NEEDS YOU": a numbered list of anything still on me. If nothing, "Needs you: nothing."
```

## Updating a repo that already adopted it

When this package changes, bring a repo's `CLAUDE.md` up to date with the box below, in a
Claude Code session opened in that repo. Adoption adds and removes nothing; an update has to
replace, because a reworded rule kept alongside its old wording is two rules. So the update
takes the template's current text for every standing behaviour and the handback contract,
keeps everything specific to the repo, and stops to show you the diff before committing.

`CLAUDE.md` carries no version line. Which package commit a repo matches is recorded in the
commit message that brought it in, where git keeps it honest, rather than in a marker in the
file that someone can bump without the change behind it.

```
=== BOX: update CLAUDE.md to the current coding discipline ===
Repo:       <absolute path of the repo>
Key:        no key, discipline update
Goal:       This repo's CLAUDE.md carries the package's current standing behaviours and
            handback contract, with every repo-specific rule kept.
Context:    The package is at <local clone path> (git pull there first). If it is not on
            this machine, read https://github.com/fightingforsidewalk/claude-code-discipline
            instead. Read only the package and this repo.
Scope IN:   CLAUDE.md; .claude/skills/ship-check/SKILL.md
Scope OUT:  no other file; no --force
Steps:      1. confirm the repo root matches Repo; pull the package; note its commit hash
            2. find the last commit here whose message names a package commit, if any, and
               diff the package from that commit to now; otherwise compare against the
               whole template
            3. for each standing behaviour and the handback contract, take the template's
               current wording; keep every rule, line note and section specific to this
               repo; where this file has its own version of a rule the template changed,
               show both and recommend one
            4. recopy skills/ship-check/SKILL.md into .claude/skills/ship-check/
            STOP: show the diff, what was replaced, what was kept as repo-specific, and
            any rule needing a choice. Wait for approval.
            5. commit both files by name with a message naming the package commit, e.g.
               "Coding sessions: update to claude-code-discipline <hash>"; push; verify the
               hash on the remote
Docs:       CLAUDE.md is the doc
Acceptance: - hash on the remote; git status --porcelain clean
            - the commit message names the package commit used
            - handback in the shape the updated CLAUDE.md specifies, Surprises numbered
=== END BOX ===
```

## After it runs

From then on, every session in that repo reads the contract automatically. The first real
handback you get back is the test: if its Surprises section says "none" on a task that
plainly had edges, send it back and ask for them.
