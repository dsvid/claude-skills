# claude-skills

A growing collection of Claude Code skills I've built for my own workflows.

## Adding a skill

1. Build it under `skills/<name>/SKILL.md` (plus any supporting files).
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

## Licence

MIT — see [LICENSE](LICENSE).
