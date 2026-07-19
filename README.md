# claude-skills

Self-authored Claude Code skills, versioned. This is **not** a backup of
`~/.claude/skills` in general — that folder mostly holds an unversioned bulk import of
external skills (different origin, not tracked here). This repo is only for skills built
by the user (with Claude's help) from scratch.

## Adding a skill

1. Build it under `skills/<name>/SKILL.md` (plus any supporting files).
2. Symlink it into the live directory so Claude Code picks it up:
   `ln -s ~/git_repos/claude-skills/skills/<name> ~/.claude/skills/<name>`
3. Commit here.

## Skills

- [`capture-learning`](skills/capture-learning/SKILL.md) — captures a concept discussed
  in a working session into that project's paired learning-resource location (cheat
  sheet entry and/or exercise), wherever and however that location is structured.
