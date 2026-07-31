# claude-skills

A growing collection of Claude Code skills I've built for my own workflows.

## Maturity

Each skill carries a `maturity:` in its `SKILL.md` frontmatter — my own signal
about how happy I am with its current state. **None of them mean frozen;
anything here may change at any rung.**

- `draft` — written, barely run. Read it for ideas; don't expect it to work.
- `testing` — in use and still changing as a result. Expect rough edges.
- `reviewed` — I've been through it and I'm happy with it: used repeatedly, and
  edits have become refinements rather than fixes.

## Adding a skill

1. Build it under `skills/<name>/SKILL.md` (plus any supporting files).
   Related skills may be grouped in a subfolder — `skills/<group>/<name>/SKILL.md`.
   The skill is then the **leaf** directory, and that is what gets copied or
   symlinked below.
2. Get it into the live directory so Claude Code picks it up, either:
   - Copy: `cp -r skills/<name> ~/.claude/skills/<name>` — use this when the
     target machine shouldn't have a live link back to this repo.
   - Symlink: `ln -s ~/git_repos/claude-skills/skills/<name> ~/.claude/skills/<name>`
     — the option for a personal machine where you want edits to flow
     straight back into git.
3. Commit here.

## Skills

- [`capture-learning`](skills/capture-learning/SKILL.md) — captures a concept discussed
  in a working session into that project's paired learning-resource location (cheat
  sheet entry and/or exercise), wherever and however that location is structured.
- [`mapping/`](skills/mapping/README.md) — a family of three: `surveyor`,
  `cartographer` and `visualiser`. Survey an unfamiliar domain, compile the
  findings into a map with provenance and resolution, and render it for a human.

## Licence

MIT — see [LICENSE](LICENSE).
