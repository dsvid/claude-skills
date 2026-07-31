---
name: capture-learning
description: Capture a concept just explained/discussed in the current session into the project's paired learning-resource location (a cheat sheet entry, and optionally an exercise) — durable, not left in chat. Use when the user hits something they don't understand in real code, asks to "add this to the cheat sheet", wants to remember an explanation for later, or a comprehension gap comes up mid-session. Not stack-specific — works for any project paired with any learning location (a language-specific playground, a consolidated docs repo, etc).
maturity: reviewed
---

# Capture Learning

Turns an in-session explanation into a durable artifact in whatever learning-resource
location is paired with the *current* project — never assume any specific stack or
repo name. Discover everything about the target from its own docs each time.

## 1. Find the paired learning-resource location

Check the current project's own memory mechanism, in order, stop at first hit:

1. If a `.beads/` dir exists here: `bd memories learning-resource` /
   `bd recall learning-resource-location`.
2. Else check this session's file-based memory for a `reference`-type entry pointing at a
   learning location.
3. Nothing found → ask the user for the path once. Save it back using whichever mechanism
   the project already uses (`bd remember learning-resource-location "<path> (<what for>)"`
   if beads, else a `reference`-type memory file) so it's only asked once per project.

This same lookup also covers the exercises-root (step 5) — the *location* is stable per
project even though *where a given capture fits inside it* is not.

## 2. Read the target location's own conventions

Read that location's `CLAUDE.md` (or `README.md` if no `CLAUDE.md`) to learn its actual
structure: cheat-sheet file names/paths, style contract, exercise/GOAL.md format, any
regen command (e.g. a TOC generator). Do not bring in assumptions from any other stack —
every convention comes from what's read here, every session.

No `CLAUDE.md`/`README.md` yet (first capture at this location) → ask the user briefly
for format preferences and suggest writing a minimal convention doc for next time, so the
next capture doesn't have to ask again.

## 3. Check existing coverage

Grep the relevant reference file(s)' headings/TOC for the concept. Already covered →
propose an amendment, not a duplicate section.

## 4. Draft the cheat-sheet addition

Follow the target location's own style contract exactly (terseness, example format,
citation format, whatever it specifies) — never apply a generic style. Use different
data/examples than that location's own exercises use, if it has that rule.

**Show the draft and stop.** Do not write until the user approves. If the cheat sheet
didn't already cover this and the wording needs iteration, iterate on the draft with the
user rather than settling for a one-shot explanation.

On approval: write it, then run the location's regen/TOC command if one exists.

## 5. Optionally propose an exercise

Ask whether this warrants a new exercise step, or a new exercise. If yes:

- **Never infer placement.** Show the existing exercise list/order and ask the user
  where this fits (which exercise it's a step of, or where a new one slots in relative to
  the others). Ordering is a curriculum judgment (prerequisites, difficulty progression)
  — not derivable from file structure.
- First exercise insertion ever at this location → also ask how mid-sequence inserts
  should be handled structurally (renumber trailing exercises vs. a suffix like
  `04b-name`), and write the answer into that location's `CLAUDE.md` so future captures
  don't re-ask.
- Scaffold only on explicit yes, using the exercise/GOAL.md conventions read in step 2.

## Non-goals

- Never write code for the user inside the target location's exercises — same prime
  directive as any tutoring context, if the target location has one (check its
  `CLAUDE.md`).
- Never invent a cheat-sheet style not already established at the target location.
- Never touch the current project's own source — this skill only writes to the paired
  learning-resource location.
