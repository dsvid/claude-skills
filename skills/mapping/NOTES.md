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

---

### 2026-08-01 — PARTIAL ANSWER: the index carries supersessions but not grounding tags, and `areas.md` is load-bearing for the visualiser

**First visualiser run happened** (`the-xemu-cartographer`, cold session,
`/map/views.md`). Two boundary facts about how the compile's output feeds the
next skill:

1. ⚠ **The "index" the visualiser actually needs is `index.md` PLUS `areas.md`.**
   The coverage view — the one this family argues is most valuable — is
   substantially a rendering of `areas.md`: resolution levels, thin-area ranking,
   the blank table. **`areas.md` is not optional cartographer output; it is the
   coverage view's data.** Both skills' contracts currently say "the index",
   singular, which understates it.
2. 🛑 **The index points at claims but does not carry their grounding tags.**
   `visualiser` §3 rule 1 ("solid source / dashed inferred / dotted docs") is
   therefore **not fully executable from a conforming index** — the tag lives in
   the record the id points at. The run worked around it by relabelling dashed as
   "inferred OR grounding not stated". **The fix belongs to `cartographer`:
   carry the grounding tag alongside the claim id.** The alternative — letting
   the visualiser open the records to look up tags — is the one move that must
   not be made.

**The relations question from the entry above is NOT resolved.** No constellation
was drawn (the human asked for coverage and architecture). Observation only: the
index states supersessions and contested pairs explicitly and **no other relation
kind**, so a constellation would likely render as a supersession chain and little
else. Still a prediction.

**New, and general to the family:** a view orthogonal to the **direction** the
map was compiled against is under-supported no matter how high-resolution the map
is. The xemu map is R3 on mechanisms and instruments, and an *architecture* view
still came out a sketch, because the direction was about evidence currency.
⇒ **Direction is not just a survey-time input; it silently bounds what can be
rendered later.** Worth stating in `README.md` where direction is introduced.

---

### 2026-08-01 — FAMILY-LEVEL CALL DEFERRED TO THE SKILLS REVIEW: should the mapping family have scripts?

**Pointer, not a restatement.** Raised by the visualiser's first run; the
evidence and the mechanism are in `visualiser/NOTES.md`, same date.

In one line: **`visualiser` §5's default output route cannot produce a navigable
diagram**, and fixing it properly means every run rewrites the same HTML viewer
boilerplate — which a script in the skill directory would solve.

🛑 **But the family currently has no scripts: three skills, six files, all
prose.** Adding one changes what this family *is* (portable, language-agnostic
instructions that need no runtime) rather than just fixing one skill. **Decide it
for the family, not for `visualiser` alone**, and decide it in the review — the
human deferred it mid-run specifically to keep s011's ordering intact.

---

## Provenance does not cover sample size — a third trap class, found by recomputation

**Source:** the-xemu-cartographer s012, 2026-08-01. Full case:
`findings/F018-2941-does-not-move-pgr1-lock-wait.md` in that repo.

The pipeline's single most valuable-looking output — the one item flagged
*"recorded, not concluded; nothing in the corpus states this"* — was **false**.
A surveyor read committed CSVs, quoted a median from run 1 of one arm against
run 1 of another, and the compile promoted it to the index's corrections table.
At n=8/arm the two distributions were indistinguishable.

**Every check in the family passed.** File path correct, arm labels correct,
`[source]` tag correct, the number really is in the file. **Provenance answers
*where a number came from*; it never answers *how many of it there were*.**

Worse: **the same extraction method produced one sound claim and one false one
in the same record.** The paired title's effect was 3–4 orders of magnitude, so
run-1-vs-run-1 was harmless there. A reviewer sanity-checking the record would
have hit the sound half first.

### What this implies for the family

- 🛑 **`surveyor` must not emit a computed statistic as a claim without n and
  spread.** A median with no denominator is a quotation, not a measurement.
  Candidate rule: *any claim whose value was computed rather than read as a
  literal string carries `n:` and a dispersion figure, or it is emitted as
  `not_determined` with the recomputation named as the resolver.*
- ⚠ **`cartographer` cannot catch this and should stop implying it can.** The
  compile adjudicates *between* records; it has no access to the underlying data.
  Promoting a single-record numeric claim into a corrections table **launders it
  into the map's most authoritative section.** Candidate rule: *corrections that
  rest on a computed statistic from one record are marked `unverified` until
  recomputed.*
- 🔎 **Open question for the review: does this family need a verification step at
  all?** Only recomputation caught it — no amount of reading would have.
  That collides directly with the open "no scripts in this family" decision
  above: **a recomputation step is the first thing that would genuinely need a
  runtime.** ⇒ **Decide the scripts question and this one together.**

### Follow-up: the escape hatch was the UNSTRUCTURED field, and the compile laundered it

Traced 2026-08-01, same session. **The surveyor's claim schema did its job. The
failure came through the one field that has no schema.**

The false claim was **not a numbered claim**. Numbered claims in that record all
carry `status / grounding / tier / provenance / accessed / volatility`, and the
fps recomputations in the sibling record all carry `n=8` and `sd`. The lock-wait
item went into the record's free-text **`unexpected:`** field, which carries
none of those, and it was honest inside its own frame — it says
*"PGR2 control **run 1** = 15,163"* and closes *"Recording, not concluding."*

**`cartographer` then promoted that free-text note into `§ D` as a topic entry
with status `current`, and into `§ J5`, the corrections table** — the map's most
authoritative section. **"run 1" and "recording, not concluding" did not
survive the promotion.**

Note the asymmetry that made it invisible: the recomputations that **confirmed**
an existing figure carried `n` and `sd`, because a prior claim was there to check
against. The one that **asserted a new effect** carried neither. **The discipline
was applied where it was least needed and dropped where it mattered.**

- 🛑 **`cartographer`: `unexpected:` / free-text fields must not be promotable to
  a topic entry or a correction at full status.** They are leads. Candidate rule:
  *content originating outside the claim schema enters the index as
  `not_determined` with the record's own hedge preserved verbatim, or not at
  all.* This is one rule and it would have caught the whole thing.
- ⚠ **This is not evidence that mapping is unreliable, and the fix is not "lower
  resolution".** ~200 pointer-claims in the same compile held up under checking.
  What failed was one claim type crossing one schema boundary.
- 🛑 **Bears directly on the planned "run the pipeline on a fresh domain" step:
  a fresh domain makes this WORSE, not better.** The only reason this was caught
  is that the underlying 65 runs were committed and could be recomputed. **On a
  fresh domain nobody can check, and the same free-text promotion would stand.**
  ⇒ **Fix the promotion rule BEFORE the fresh-domain run**, or that run cannot
  distinguish a good map from a fluent one.

### FIXED 2026-08-01 — and here is the regression test

Applied to all three skills. **Three independent catches**, deliberately, because
the defect passed every check the family had:

| # | Skill | Rule |
|---|---|---|
| 1 | `surveyor` §5 | **A derived statistic carries its `n` and its spread, in the statement.** Never describe a subset as its population. No `n` ⇒ `not_determined` |
| 2 | `surveyor` rule 4 | **`unexpected` takes leads, never falsifiable assertions or numbers.** Rule 4 previously *routed* forming conclusions there — it was the pipe feeding the hole |
| 3 | `surveyor` schema | `unexpected` annotated **LEADS ONLY, not promotable** |
| 4 | `cartographer` §2 | **Only `claims:` may become an index entry.** Free text → a lead at most, **hedge preserved verbatim**. Corrections resting on free text are `unverified` |
| 5 | `cartographer` §2 | **A comparative statistic with no `n` cannot be promoted.** The compile does not hold the data, so it must not imply it checked |
| 6 | `cartographer` §7 | Read-back gate: **trace every entry back to its field**; mark the entry you would be most embarrassed by if the human recomputed it |
| 7 | `cartographer` rule 5b | **Compiling never upgrades a claim's standing.** The index inherits authority; it cannot confer it |
| 8 | `visualiser` §6 | Never render an `unverified` or `n`-less figure as a bare number — a diagram strips every qualifier the index attached |

**Regression test — the original case, run against the new rules:**

1. Surveyor writes it as free text → **rule 4 blocks it**: falsifiable + numeric,
   so it must go to `claims:`.
2. In `claims:` → **the `n` rule fires**: it is n=1, and *"never describe a subset
   as its population"* catches `PGR1 control = 31` (that is run 1). ⇒
   `not_determined`, resolved by recomputation.
3. If it escaped both, **cartographer §2 blocks promotion** to § D / § J5; it
   lands as a lead carrying *"recording, not concluding"* verbatim.
4. If it escaped that, **the read-back gate asks which correction rests on
   `unexpected:`** — § J5 is the answer.

✅ **Caught four times over.** The original was caught zero times.

### ⚠ The cost, stated honestly

**The same record's PGR2 migration observation was also free text, and it is
true and valuable.** Under the new rules it is demoted to a lead needing
recomputation — which is exactly the round-trip s012 had to do anyway.

**So the fix trades a true finding's authority for a false one's suppression.**
That is the right trade here (a lead still surfaces; a laundered claim gets
believed), but it is a real cost and it will make some compiles feel thinner.
🛑 **Watch for the failure mode where everything interesting becomes a lead and
the index goes bland.** If that shows up, the answer is a recomputation step —
not relaxing the rule. ⇒ ties back to the open scripts question above.

---

## The bigger hole, found while fixing the first: `not_determined` does not survive the compile

**Measured on the first real compile, 2026-08-01. 49 `not_determined` items went
in. About 4 came out.**

The index had bubbling registers for **contested**, **staleness** and
**corrections** — and **none for open questions**. So the surveyor collected them
diligently, per its own rule 1, and the cartographer evaporated ~45 of them.

🛑 **This is a bigger defect than the n=1 promotion that led me to it.** That one
put a wrong claim in the map; this one **removes the map's account of what it
does not know** — and a map that has quietly lost its own gaps is the exact
artefact this family exists to prevent.

**Why it happens structurally, not by carelessness:** the index is keyed
*topic → current claim*. A `not_determined` names something **absent**. There is
no current claim for it to attach to, so it has nothing to bind to in a
topic-keyed structure and falls out silently. **Every other record field has a
natural home in the index; this one does not.** ⇒ it needs an explicit register
or it is lost by construction.

**The detail that makes the case:** among the dropped items was a record's doubt
about *how a dispersion figure was defined* (`"what statistic do STATUS.md/F016
mean by 'the spread'?"`). It was dropped in the **same compile** that promoted a
comparison with no `n`. **The pipeline had already noticed the adjacent problem
and the compile threw the notice away.**

### Fix applied

- `cartographer` **§5b (new)**: every `not_determined` bubbles to an
  **open-questions register** — question, record, `resolved_by` verbatim, cost —
  ranked cheapest first. Dropping one requires naming it in the hand-over.
  Questions the *compile itself* creates (`unverified`, demoted leads) go there
  too, or the compiler's own caution evaporates with it.
- `cartographer` **§7 read-back gate**: **count them in, count them out.** If the
  register is smaller than the input, name every missing one or restore it.
  By counting, not by impression.
- `cartographer` **rule 3**: "never smooth over a gap" now explicitly covers
  compile-time drops — *a `not_determined` recorded in a survey and dropped at
  compile is smoothed over more thoroughly than one never written, because
  someone did the work of noticing and the map now implies nobody did.*

### The design position this settles

Considered and **rejected**: a bundled recomputation script. Recomputation is
irreducibly domain-specific — CSVs here, SQL elsewhere, log parsing elsewhere
again — so a generic one cannot exist, and it would sit surveyor-side anyway
(the cartographer never holds the data). **The family stays prose-only.**

✅ **The answer is not to verify more; it is to state clearly what was not
verified, and bubble it where someone walks into it.** That is cheaper, it is
domain-agnostic, and it is the thing the pipeline was already 90% doing.

---

## Scoring the skills themselves → `EVALUATION.md`

**Moved 2026-08-01.** The pre-registered criteria for the fresh-domain
generality run — conservation, precision, confabulation, oracle, usefulness,
plus what would kill the family rather than tune it — now live in
**`skills/mapping/EVALUATION.md`**, together with the scores of each run.

🛑 **Not duplicated here.** NOTES records *what we learned building and running
these*; `EVALUATION.md` records *how we decide whether they work*. Content lives
in one place or neither is authoritative.

## Reach is a **pass**, not just a scope (human's refinement, 2026-08-01)

*"It might be both, or I'd run an inward survey first and then an outward survey
second — and both should be additive and feed cleanly into the cartographer."*

⇒ Applied across **both** skills, because it is a pipeline property, not one
skill's:

| Skill | Change |
|---|---|
| `surveyor` §0 | Reach can be run in passes. A later pass **adds**; it never replaces. Say which pass this is, in the hand-over and the records |
| `surveyor` §2 | The index tells you a **third** thing now: what was **never in scope**. An area missing from an `inward`-only map is unmapped, not empty |
| `cartographer` §5 | Record a **pass log** — which reach was compiled and when. Distinguish **thin** (looked at, shallowly → re-survey) from **unsurveyed by reach** (nobody looked → new pass) |

🛑 **Why this needed a change at all — the failure it prevents is silent.** An
`inward`-only compile is **complete, internally consistent, and reads as though
the subject stands alone.** The absent ecosystem leaves *no trace*: no record,
no resolution, no `not_determined`, nothing for the skip list to catch. **It is
the one gap in this format that is invisible from the inside**, and the pass log
is the only thing that makes it visible.

✅ **The additive half needed no machinery** — claims are append-only and the
index is topic-keyed, so an outward pass adds topics beside the inward ones.
**The rule added is a prohibition, not a mechanism:** do not re-survey the
subject to "re-integrate" it, and do not re-open settled inward entries because
a new pass arrived.

⚠ **Cost, tracked honestly: `surveyor/SKILL.md` 291 → ~450 lines in one day.**
That is a lot of spec for one scored run's worth of evidence. **Standing note:
after run 3, consolidate before adding anything further** — several of today's
additions overlap (§0 reach, §3 outward playbook, §7 gate all bear on breadth)
and could likely be one shorter section.
