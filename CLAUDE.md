# claude-skills

A personal collection of Claude Code skills. **This repo is public** — assume
anything committed here is read by strangers.

`README.md` is the front door for those strangers: keep it short. This file is
the working convention, and it is where new rules go.

## 🛑 Never push. The human does all outward-facing git.

Local commits are the normal way to work. **Pushing, opening PRs, and anything
else that leaves the machine is the human's to run** — write it up and hand over
the command.

## Layout

```
skills/<name>/SKILL.md              # a standalone skill
skills/<group>/<name>/SKILL.md      # a grouped skill — the LEAF is the skill
```

**A group is either filing or a family. Say which, in the group's `README.md`.**

- **Filing** — related things stored together. They do not need each other.
- **Family** — the skills share a contract and feed each other's output. Take
  one out on its own and you have a piece of a pipeline, not a tool.
  See `skills/mapping/` for the worked example.

### 🛑 A family's contract must live INSIDE each skill, not in the group folder

**Only `SKILL.md` and its own bundled files are loaded when a skill runs.** The
group `README.md` is not — and under the `cp -r` install below, the group folder
is not even present, so a reference to `../` dangles. A contract kept only in
the group folder is invisible at exactly the moment it matters.

So:

- **Each skill states its own input and output contract inline**, precisely
  enough to be honoured without reading anything else. Adjacent skills' contracts
  must agree; that overlap is the only place drift can happen, so keep it small
  and explicit.
- **Each skill declares the format version it speaks** (e.g. `map format v1`).
  Meeting an artefact that declares another version is a **stop**, not a
  best-effort write.
- **If the target location has its own spec, it wins** — read it on every run,
  the way `capture-learning` reads its paired location's conventions.
- **The group `README.md` is for humans**: what the family is, why, the canonical
  spec, and the place a change gets reconciled across all of them. It is
  documentation and a review checkpoint — never a runtime dependency.

## What a skill actually sees when it runs

Observed 2026-07-31 by invoking `capture-learning` and reading what arrived:

```
Base directory for this skill: /Users/davidlaroche/.claude/skills/<name>
<the body of SKILL.md, from the first heading>
ARGUMENTS: <whatever was passed>
```

- **Frontmatter is stripped.** `name`, `description` and `maturity` are *not*
  visible at runtime — they are for the loader and for humans. Do not design a
  runtime behaviour that depends on reading them.
- **The base directory is announced**, so a skill can read its own bundled files.
- **Nothing outside the skill directory is loaded** — not the group `README.md`,
  not this file.

Under a symlink install, edits to a skill **body** take effect on the next
invocation. The **skills list** (names and descriptions) is assembled at session
start, so a changed `description` needs a new session before routing sees it.

## Frontmatter

```yaml
---
name: skill-name
description: What it does. Use when [specific triggers].
maturity: draft | testing | approved
---
```

- **`description` is doing routing work.** It is the only thing an agent sees
  when deciding whether to load the skill. Keep it to capability + triggers.
  **Never put status, caveats, or version notes in it** — they compete with the
  matching it exists for.
- **`maturity` is the human's own signal**, defined in `README.md`.
  🛑 **Only the human moves a skill up a rung.** Do not promote a skill because
  it looks finished; the rung records their confidence, not yours.
  - `draft` — written, barely run.
  - `testing` — in use and still changing as a result.
  - `reviewed` — been through it and happy with it.
  - **None of them mean frozen.** A skill at any rung may still change.

**A skill at `testing` keeps a `NOTES.md` beside its `SKILL.md`.** Append
friction as it happens — what was ambiguous, what had to be worked around, what
the human corrected. One line each; do not interrogate the user about how it
went. That file is the evidence for the promotion decision, and without it the
rung is a feeling.

## Skills carry method, not domain content

A skill should work wherever it is pointed. **Domain conventions belong to the
domain** — read them from the target location's own `CLAUDE.md`/`README.md` on
every run, as `capture-learning` does, rather than baking a vocabulary into the
skill.

⚠ **The recurring failure is promoting a vivid domain-specific lesson into
general skill content because it is the best material available.** If a rule
would be meaningless in a different domain, it is not skill content.

## Installing

Discovery is by the entry in `~/.claude/skills/<name>` — copy or symlink, and
the source can be nested however this repo likes:

```bash
# personal machine — edits flow back to git
ln -s ~/git_repos/claude-skills/skills/<group>/<name> ~/.claude/skills/<name>

# elsewhere — no live link back
cp -r skills/<group>/<name> ~/.claude/skills/<name>
```

**Not installing is how a draft stays out of the way.** An uninstalled skill is
not loaded, not listed, and cannot fire by accident.

## Do not edit third-party skills

`~/.agents/skills/` is installed from other people's repos (`mattpocock/skills`)
and managed by `~/.agents/.skill-lock.json` with per-folder hashes. **Local
edits there are overwritten on update.** Anything worth keeping gets written
here instead.

## Public-repo hygiene

Worked examples are the usual leak. Before committing an example, check it does
not carry:

- names of real people alongside metrics about them,
- private material from whatever project the skill was derived from,
- credentials, internal URLs, or host-specific paths that are not obviously
  generic.

**A skill's examples are the part most likely to be read and copied.** Prefer an
invented example to a real one that needs redacting.
