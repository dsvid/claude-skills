# mapping — a family of skills

**This is a family, not a category.** The folders under `skills/` elsewhere group
skills that merely file together; these three are *coupled*. One's output is the
next one's input, and they share a single artefact — **the map**.

If you extract one of these on its own, you are taking a piece of a pipeline.

```
direction ──► surveyor ──► survey records ──► cartographer ──► map + index ──► visualiser ──► diagrams
                  ▲                                  │
                  └────────── re-survey ◄────────────┘
```

| Skill | Does | Reads | Writes | Maturity |
|---|---|---|---|---|
| `surveyor` | Gathers: what is this thing for, what related things exist, how do they relate | the world | survey records | `draft` |
| `cartographer` | Compiles records into a map; owns supersession and resolution | survey records | map + **index** | `draft` |
| `visualiser` | Renders the map for a **human** | **index only** | diagrams | `draft` |

A fourth, a **deep-diver** (hypothesise → falsify → measure), is deliberately
deferred. Its scope is undefined *because* its boundary is with the map; build
and run these three first and the boundary becomes an observation rather than a
guess.

## What this document is — and is not

**This is the canonical spec, for humans.** It is where the contract is agreed,
compared across the three, and changed.

🛑 **It is not loaded at runtime.** When a skill runs, only its own `SKILL.md`
and bundled files are read — and under a `cp -r` install this folder is not even
present. So **each skill restates the part of the contract it must honour,
inline**, and declares the format version it speaks. A mismatch is a stop.

⇒ **Changing the spec here changes nothing by itself.** Propagate it into the
three `SKILL.md` files and bump the version, or the map and the skills quietly
disagree.

---

## The map

The map is **not a document**. It is three things with different mutability
rules, and confusing them is the failure mode this design exists to prevent.

| Part | Mutability | Purpose |
|---|---|---|
| **Claims** | **Append-only, immutable** | What was found, with provenance. Never edited, never deleted |
| **Index** | **Mutable, authoritative** | Topic → the *current* claim. The **mandatory entry point** |
| **Areas** | Mutable | The territory, each with a recorded **resolution** |

### 🛑 The index is the entry point. Always.

Never answer a question by searching the claims directly. Searching returns
every claim that ever matched, with no notion of which is current — a long map
supersedes itself, and a stale claim with correct provenance passes every other
check you could apply to it.

Look up the topic in the index; the index names the current claim. If the index
has no entry for a topic, that is a finding: the area is unmapped, and saying so
is the correct answer.

**Supersession is an index operation.** A later claim that overturns an earlier
one does not edit or delete it — the claim stays, the *index entry* is
repointed. The reasoning trail survives; only the answer moves.

---

## Every claim carries three independent attributes

They are orthogonal. Each catches a failure the other two miss.

### 1. Grounding — how it is known

- `[source]` — read directly from the thing itself.
- `[docs]` — stated in a README, wiki, or comment. May be stale; prefer source
  where both exist, and **record the disagreement, because it is itself a
  finding**.
- `[inferred]` — deduced. **Must show the reasoning.** These are the risk
  register; when something turns out wrong, it is almost always here.

**Never smooth over a gap.** Write "not determined" and say what would resolve
it. A fluent, unsourced map is worse than no map, because it will be believed.

### 2. Provenance tier — how re-checkable the source is

Cumulative: each tier includes everything below it. Record the highest that
honestly applies.

| Tier | Name | Carries | Someone else can… |
|---|---|---|---|
| **T4** | **Immutable** | An upstream content-addressed ID — git SHA, DOI, archive URL | …verify independently, forever |
| **T3** | **Snapshot** | Our own stored copy of the response | …see exactly what we saw, if they trust our copy |
| **T2** | **Addressable** | How to get back to it — a URL, or a **verbatim query** | …re-run it and observe drift |
| **T1** | **Dated** | What was consulted, and when | …know how old it is, and little else |
| **T0** | **Asserted** | Nothing | …only take our word for it |

**T0 must be visible, not hidden.** It is legitimate for things like a
colleague's recollection — but it must be *marked*, so nobody later mistakes it
for a checked fact.

#### T2 splits: a document is not a query

- A **document** has a stable identifier and mutable content. Re-fetching the
  URL gets you the same thing, possibly updated. Record the URL.
- A **query** has no such identifier — re-running it returns a different
  *population*, not an updated document. **Record the query verbatim**, because
  a slightly different query silently yields a different answer and nobody can
  tell which was run. For a query, the query text is part of the claim, not part
  of the citation.

#### Human and verbal sources

No URL and no hash, but they are not unprovenanced. Record **who**, **when**,
**what exactly was said**, and **whether it has been tested**. Domain intuition
from a practitioner is a measurement channel, not colour; log it with the same
discipline as any other reading, and treat it as a hypothesis rather than a fact
until something checks it.

#### Derived claims

A claim computed from other claims — a statistic over stored data, a diff
between two sources — is pinned by **the transformation plus its inputs**.
Record the script (or the exact operation) and the input artefacts. It is
reproducible only if both are recorded.

### 3. Volatility — how fast it decays

The tier tells you how to re-check something; volatility tells you *whether you
need to*. Two claims from the same source on the same day can differ wildly:

- `static` — a closed historical record. Will read the same in five years.
- `slow` — changes over months.
- `live` — changes daily. **Re-check before quoting.**

Every claim records **when it was accessed**, in every tier including T4.

---

## Resolution — the map is drawn at different depths in different places

Map **broadly before deeply**. Establish a coarse, accurate frame over the whole
territory first, then increase resolution where it matters. Detail bought before
the frame exists is detail positioned against nothing — and it is the most
common way this process wastes weeks.

| Level | Name | What is known |
|---|---|---|
| **R0** | **Named** | It exists. Name, one-line description, where it lives |
| **R1** | **Sketched** | What it is for, and a **guessed** relation to the rest. `[inferred]` expected |
| **R2** | **Surveyed** | Purpose and relations verified against source or docs |
| **R3** | **Detailed** | Internals mapped — structure, parts, how it actually works |

**Resolution is recorded per area, and it is what makes low-resolution honest.**
A guessed relation is perfectly acceptable at R1 and dishonest at R3 — the only
difference is whether the resolution is written down. An unmarked guess is the
fluent unsourced map arriving by the back door.

**Raising resolution is an ordinary re-run**, not a rewrite. The area keeps its
identity; new claims are appended, the index is repointed, the level goes up.

**Blank is drawn as blank.** An unmapped area is drawn with its boundary and
left empty. Omitting it entirely reads as "there is nothing there", which is a
claim nobody made.

---

## Rules all three skills obey

1. **The index is the entry point.** Never answer from a search of the claims.
2. **Claims are append-only.** Supersede by repointing the index; never edit or
   delete a claim.
3. **Every claim carries grounding, tier, access date, and its area's
   resolution.** No exceptions, including for the skills' own assertions about
   their tooling — infrastructure prose gets trusted exactly as much as a
   finding.
4. **Never smooth over a gap.** "Not determined" plus what would resolve it.
5. **Blank is drawn as blank.**
6. **A change of direction re-fires the survey.** The map was surveyed to answer
   a question; change the question and the map is no longer aimed at it. Go back
   to survey, not forward.
7. **Exhaust what you already have before gathering more.** Ask what the
   existing records can still be asked. This is not thrift — re-reading existing
   material outperforms new collection often enough that it deserves to be a
   gate, not a suggestion.
8. **Read forward before citing.** If a topic has several claims, the index
   names the current one. Trust the index over your search results.

---

## The direction

**A survey without a direction maps forever.** Every run of this pipeline starts
with a written direction: *what question is this map being drawn to answer?*

It is not decoration. It is:

- **the stopping rule** — the map is done when it answers the direction, not
  when the territory is exhausted (it never is);
- **what each sub-agent is given** — surveyors fan out and start cold, so the
  direction is what makes parallel work coherent;
- **what resolution decisions are made against** — an area gets more detail
  because the direction needs it, not because it is interesting.

Record the direction with the map. When it changes, record that too, and
**re-run the survey against the new one** — an old map aimed at an old question
is the most confidently wrong artefact this process can produce.

---

## Per-domain conventions live with the map, not here

These skills carry **no domain content**. What a map of a codebase contains
differs from what a map of an organisation or a document corpus contains — and
that belongs to the mapped domain, recorded in its own `CLAUDE.md` (or
`README.md`) and re-read on every run.

This document defines the **format**; the domain defines the **content**. Do not
bake a domain's vocabulary into a skill.

---

## Known weaknesses of this design

Written before the first run, so they are not discovered as pleasant surprises.

- **The seams are duplicated on purpose.** The record schema appears in
  `surveyor` and `cartographer`; the index schema in `cartographer` and
  `visualiser`. A skill cannot read this file at runtime, so the duplication is
  the price of surviving a `cp -r` install. **The version string is the only
  thing preventing silent drift — bump it whenever a seam changes, and
  propagate.**
- **The pipeline is a loop, not a line.** A deeper pass re-enters at `surveyor`;
  a changed direction re-fires the whole thing. Anything that treats it as
  one-way will produce a map aimed at a question nobody is asking any more.
- **Resolution is the least-tested idea here.** R0–R3 are plausible and unproven;
  the boundaries between them are judgement calls that different runs may make
  differently.
- **`contested` is more work than picking a winner**, and picking a winner
  always feels more helpful. Expect pressure against it.
- **Sub-agent fan-out flattens surprises.** The `unexpected` field catches some of
  it, and it is a weak mitigation — a cold sub-agent may not recognise a surprise
  as such. ⚠ **And `unexpected` is leads-only** (2026-08-01): it is not a channel
  for findings, because nothing in it has passed the claim schema.

Each skill's `NOTES.md` carries its own pre-registered open questions, and
`cartographer/NOTES.md` holds the four-case regression test for the index.
**`EVALUATION.md` is how we decide whether any of it works** — the five tests,
what "good" means per skill, and the scores of every run so far.

## Status

All three are `draft`. **Run three times, on one domain**
(`the-xemu-cartographer`, 2026-07-31 → 08-01).

🛑 **The first real run failed two of the five tests in `EVALUATION.md`**: the
compile lost ~45 of 49 open questions, and it promoted a derived statistic
computed over n=1 into the map's corrections table. **Both were spec defects, both
are fixed, and neither fix has been validated by a run.**

⚠ **Nothing here has been validated against a domain other than the one it was
derived from.** Running them on that domain tests the *mechanics* — whether the
index resolves, whether the record schema is sufficient, whether the render
stays honest. It cannot test whether the method generalises, and a clean run
should not be read as evidence that it does.

✅ **What that one domain did establish:** the pipeline found 11 dead-but-live
claims and one live-but-dead claim that months of human reading had missed. **The
method finds real things.** What is unproven is that it finds them anywhere else,
and that it stops asserting things it has not checked.

See the root `README.md` for what the maturity rungs mean.
