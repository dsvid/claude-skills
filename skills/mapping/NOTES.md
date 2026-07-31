# mapping — family-level notes

Friction and design observations that belong to **the family**, not to any one
skill. Per-skill observations live in `surveyor/NOTES.md`,
`cartographer/NOTES.md`, `visualiser/NOTES.md`.

Use this file when an observation is about **the pipeline** — ordering, handoff,
the shared map artefact, or the boundary between two skills.

---

## Friction log

### 2026-08-01 — RUN THE MAP UPFRONT, NOT AS ARCHAEOLOGY (the human, after the first full survey→compile)

> "When running the survey and cartography skills against future repos we'll note
> to do it upfront now that it's defined, then any further findings can go
> against it."

**Context:** first full pipeline run was against `the-xemu-cartographer`, a
~10-session analysis repo whose corpus had already gone stale. Two sessions
(survey, then compile) produced **zero new domain evidence**. What they produced
was excavation: 11 dead-but-live-looking claims and 6 contested ones from the
survey, 6 more corrections from the compile.

**The point:** that work was archaeology, not cartography. Run upfront, the same
structure costs almost nothing:

- The index starts empty and grows one entry per topic as findings arrive.
- **Supersession becomes a one-line repoint at the moment it happens**, done by
  the agent who has the context to know it happened — not reconstructed later by
  someone reading a 3,061-line document forward.
- The "read forward before citing" discipline becomes unnecessary, because the
  index already says what is current.

⇒ **The expense is in retrofitting, not in maintaining.** That is a stronger
argument for the pipeline than any argument for either skill alone.

#### The sharper reason, found by the first compile

Read-forward has **no defence against a misaimed refutation.** In that repo, a
document declared a claim refuted while naming the wrong claim number; the live
claim it actually named has sat marked-dead ever since, in the file the repo's
own session protocol opens every session. A misnumbered refutation passes every
provenance check, because the *refutation* is well-sourced — its *target* is
wrong.

Retrofitting catches these only unreliably, because the excavator must
reconstruct which claim a refutation *meant*. **Maintained live, the collision
surfaces the day it is written**, while the author still knows.

#### 🛑 The caution that must travel with this

All three skills were derived from **that one repo**, with its corpus, its
failure mode and its epistemic rules in front of the author while writing.
Working there is not evidence of generality.

⇒ **Do the upfront run on the next domain as a TEST, not as settled practice.**
Pre-register the falsifier: *if maintaining the index upfront costs more per
session than it saves, or if the index goes stale the same way the incumbent
status document did, the ordering claim is wrong.*

---

### 2026-08-01 — "feeds" is an ACTION, not a configuration setting

`cartographer` §1 makes the run ask whether its index **replaces / feeds /
beside** the incumbent entry point. The first run asked, got **feeds**, and
recorded it — **and then nothing made anyone perform the feed.**

The compile found six corrections it was forbidden to apply (rule 7: never edit
the territory) and had to invent a place to put them. They ended up in a section
of the index that **nothing in the domain's own open protocol points at**.

🛑 **A map that can only report, never reconcile, becomes a second index** — the
exact failure §1 exists to prevent, reached by *obeying* the skill.

**Two contract changes this implies:**

1. **Name the reconciliation pass as a required follow-up**, with its own
   session, rather than leaving it as an implication of the entry-point answer.
   `feeds` and `beside` both create an obligation; the skill should say so.
2. **Whatever is compiled must be reachable from the incumbent entry point on
   day one.** Writing a pointer *into* the incumbent should be part of the first
   compile — the one permitted write to the territory, or an explicit hand-back
   to the human. Otherwise the open protocol routes every future session around
   the map.

**Also worth a section in the skill's output format:** "corrections owed to the
territory". Rule 7 guarantees a compile will find staleness it cannot fix. With
nowhere structured to put it, that finding either leaks into the territory or
dissolves into prose.

---

### 2026-08-01 — OPEN BOUNDARY QUESTION: does the index carry relations, or only statuses?

**Unresolved, flagged before the visualiser's first run.** Full pre-registration:
`visualiser/NOTES.md`, same date.

The pipeline splits one artefact across two contracts:

- **`cartographer`** writes an index keyed by *topic → current claim*, carrying
  status, resolution, volatility, supersession and contested pairs.
- **`visualiser`** reads **the index only** — deliberately, so a diagram can
  never assert more than the map does.
- **But the `relations:` blocks live in the survey RECORDS** (`refutes`,
  `depends on`, `measures`, `supersedes`), and the first compile did not lift
  them into the index.

⇒ **The constellation view may be unbuildable from a conforming index.** If so
the fault is not the visualiser's: **the cartographer's output format is missing
a field the next skill in the pipeline needs.**

**This is the first case found of the family's contracts not composing.** Worth
watching for others: each skill's contract was written against the map, not
against the next skill's input requirements.

🛑 **Do not resolve it by widening the visualiser's read permission.** That
trades the one structural guarantee in the pipeline for a nicer picture.
