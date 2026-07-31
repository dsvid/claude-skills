# cartographer — notes

Append friction as it happens: what was ambiguous, what had to be worked around,
what the human corrected. One line each. This is the evidence for promoting the
maturity rung.

## The regression test — run this against a real corpus

The index design came from four **known failures in an existing body of
findings**. A candidate index is good enough to start if it handles all four,
and will fail in practice if it does not. These are from the corpus the skill
was derived from; substitute equivalents in any corpus with real history.

1. **A stale claim with correct provenance.** A long document whose later claims
   withdraw its earlier ones. Ask the index the topic; it must return the
   **last** claim on it, not the first one a search would hit.
   *Tests: keyed at claim level, and supersession actually repoints.*
2. **A genuinely contested topic.** A claim marked refuted in one document and
   restored in another, where the honest status is unresolved. The index must be
   able to say `contested` and name both.
   *Tests: that it does not manufacture a current answer. **The case most likely
   to break a naive schema.***
3. **Two claims, one source, one day, different decay.** A live count and a
   closed historical record gathered together. If `volatility` attaches to the
   source rather than the claim, the schema is wrong.
4. **Supersession across documents.** A measurement in one finding refuting an
   inferred claim in another. Tests that repointing is not scoped to a file.

## Open questions, written before the first run

- **Is "the question someone would actually ask" a stable key?** Topics may be
  named differently on each run, silently creating duplicate entries for one
  question. If so the index needs a controlled vocabulary, which is a much
  bigger change.
- **Does `contested` get used, or avoided?** It is more work than picking a
  winner, and picking a winner always feels more helpful. Watch for entries
  marked `current` where the evidence genuinely does not settle it.
- **Does the read-back gate catch anything, or is it ceremony?** It exists
  because a followed protocol still produced a bad handoff. If three runs pass
  it with nothing found, either it works or it is being performed rather than
  done — distinguish those before trusting it.
- **Is "grounding, then tier, then recency" right?** It is a judgement, not a
  measured rule. Look for a case where it produces an answer that feels wrong.
- **Does the thin-areas list get produced and used?** It is the main output that
  tells the human whether to re-run the surveyor.

## Friction log

<!-- newest first; date each entry -->

### 2026-07-31 — pre-run: the write-boundary gap (raised by the human, before the first run)

**The human asked for a prompt telling the cartographer not to update
`STATUS.md`, then asked the right follow-up: *"is that something we'll have to be
careful about every time, or is it just for this repo?"***

**It is general, and needing to say it by hand is the gap.**

**The collision, stated generally:** the index declares itself *"the mandatory
entry point"*. **Most established domains already have one** — a `STATUS.md`, a
README, a wiki homepage, an ADR index, a runbook. So on arrival the cartographer
meets an incumbent entry point it was not told about, and **two authoritative
entry points is the exact failure this skill exists to prevent.** It will be
tempted to fix that by absorbing or rewriting the incumbent.

⇒ **§1 must ask a question it currently does not:** is there already a
human-facing entry point here, and is the index **replacing** it, **feeding** it,
or **living beside** it? All three are legitimate; silently picking one is not.

**`capture-learning` already solves this and the mapping skills copied only half
of it.** It has two pieces we lack:

1. **A resolved *and persisted* target** (`capture-learning` §1): check
   `CLAUDE.md` → `bd recall learning-resource-location` → else ask, **and write
   the answer back with `bd remember`**. Ours says "ask where the map lives" —
   which re-asks every single run and never learns.
2. **An explicit negative write boundary** (`capture-learning` line 76):
   > *"Never touch the current project's own source — this skill only writes to
   > the paired learning-resource location."*

**(2) is the whole answer to the human's question.** With that rule, "don't touch
`STATUS.md`" needs no prompt in any repo, because the skill only ever writes to
the map. **Without it, every run needs a hand-written guard, in every domain** —
and the one time it is forgotten, the skill edits the artefact the human relies
on. `surveyor` and `visualiser` need the same rule; `visualiser` most of all,
since renders are the easiest thing to scatter into a repo.

⚠ **Not implemented — deliberately.** Doing it now would change the skill
between drafting it and first running it, and the first run is the only clean
observation we get of the drafted version. **Watch what it actually does about
`STATUS.md` on run 1**, then implement. If it asks unprompted, the gap is smaller
than it looks.
