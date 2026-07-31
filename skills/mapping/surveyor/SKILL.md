---
name: surveyor
description: Survey an unfamiliar domain against a written direction — what a thing is for, what related things exist, what they do, and how they relate — and emit provenance-carrying survey records for the cartographer to compile. Use when arriving at an unfamiliar codebase, ecosystem, organisation or document corpus and needing to establish what is there before analysing it; when asked to map, survey, scope or get the lay of the land; or when an existing map needs an area re-surveyed at higher resolution.
maturity: draft
---

# Surveyor

**Speaks map format v1.** Emits survey records for `cartographer`. Does not draw
the map and does not diagnose anything.

Your job is to find out **what is there**, broadly before deeply, and to record
it so that someone who was not present can check every word of it.

## 0. The direction — no direction, no survey

**A survey without a direction maps forever.** Before anything else, get one
sentence in writing: *what question is this map being drawn to answer?*

If the user has not given one, ask for it. If they are unsure, propose one from
what they have said and get it confirmed. **Do not start on a vague brief** —
the direction is the stopping rule, the fan-out instruction, and the thing every
resolution decision is made against.

Record the direction with the records you emit. **If the direction changes
later, the survey re-fires** — a map aimed at an old question is worse than no
map, because it looks current.

## 1. Find the map, and read its conventions

Ask where the map lives, or find it: a `map/` directory, an `_inventory`, an
existing index. If there is none, ask where it should go.

**Read that location's own `CLAUDE.md` or `README.md` every run.** Domain
conventions belong to the domain, not to this skill — what a map of a codebase
contains differs from a map of an organisation. Never import conventions from
another run.

**If the map declares a format version other than v1, stop and say so.** Do not
write records it cannot read.

## 2. Ask what is already known before gathering anything

Check the existing index for the areas your direction touches, and note their
resolution. **Re-surveying ground already covered at sufficient resolution is
the single most common waste.**

Then ask the two questions that repeatedly turn out to matter:

- **What can existing material still be asked?** Notes, captures, logs and
  records already collected often answer the direction without new gathering.
  Exhaust them first — this is a gate, not a suggestion.
- **Has someone already answered this?** Search for prior art aimed at *the
  direction*, not at the subject in general. Issue trackers, open PRs, prior
  write-ups, published work. Finding that the question is already answered is a
  successful outcome, not a wasted survey.

## 3. Fan out

For anything more than a couple of subjects, dispatch **one sub-agent per
subject**. They work independently and return a bounded record; you stay
uncluttered enough to compile.

**Every sub-agent gets the direction verbatim.** They start cold — the direction
is what makes their independent work coherent, and without it they will report
what is interesting rather than what is relevant.

Give each one the record schema below and require it back in that shape.

**Follow references; do not browse.** Survey what the material actually points
at — what the code imports, what the docs cite, what the issues link. A
plausible-sounding name in a listing is not a reference.

## 4. Breadth before depth

Establish a coarse, accurate frame over the whole territory first, then increase
resolution where the direction needs it. Detail bought before the frame exists
is detail positioned against nothing.

| Level | Name | What is known |
|---|---|---|
| **R0** | Named | It exists. Name, one-line description, where it lives |
| **R1** | Sketched | What it is for, and a **guessed** relation to the rest |
| **R2** | Surveyed | Purpose and relations verified against source or docs |
| **R3** | Detailed | Internals mapped — structure, parts, how it works |

**R0 and R1 are legitimate outputs, not failures** — provided the level is
recorded. A guessed relation is honest at R1 and dishonest at R3, and the only
difference is whether you wrote the level down.

## 5. The survey record — map format v1

One record per subject. This is the contract `cartographer` consumes; keep the
field names.

```yaml
record: <subject-slug>
direction: <the one-sentence direction this was surveyed against>
surveyed: <YYYY-MM-DD>
resolution: R0 | R1 | R2 | R3
subject:
  name: <what it is called>
  kind: <repo | document | service | team | dataset | …>
  location: <url, path, or where it lives>
  summary: <one line — what it is>
purpose: <what it is for, one short paragraph>
relations:
  - to: <other subject>
    nature: <depends on | consumes | supersedes | forked from | discusses | …>
    grounding: source | docs | inferred
    reasoning: <required when grounding is inferred>
claims:
  - statement: <a falsifiable sentence>
    grounding: source | docs | inferred
    tier: T0 | T1 | T2 | T3 | T4
    provenance: <path+lines, URL, verbatim query, snapshot path, or who said it>
    accessed: <YYYY-MM-DD>
    volatility: static | slow | live
not_determined:
  - question: <what could not be established>
    resolved_by: <what would settle it>
unexpected: <free text — anything noticed that the direction did not ask for>
```

### Provenance tiers — record the highest that honestly applies

| Tier | Carries |
|---|---|
| **T4** Immutable | An upstream content-addressed id — git SHA, DOI, archive URL |
| **T3** Snapshot | Our own stored copy of the response, with its path |
| **T2** Addressable | A URL, **or a verbatim query** |
| **T1** Dated | What was consulted and when, nothing re-fetchable |
| **T0** Asserted | Nothing. Legitimate for recollection — **must be marked** |

**A document is not a query.** A URL returns the same thing, updated. A query
returns a different *population* — so record the query **verbatim**, because a
slightly different one silently yields a different answer and nobody can tell
which was run.

**Human sources are not unprovenanced**: record who, when, what exactly was
said, and whether it has been tested. Treat it as a hypothesis, and say so.

**Derived claims** are pinned by the transformation plus its inputs — record the
operation and the input artefacts, or it is not reproducible.

## 6. Rules

1. **Never smooth over a gap.** Write it into `not_determined` with what would
   resolve it. A fluent, unsourced map is worse than no map, because it will be
   believed.
2. **Every claim carries grounding, tier, accessed date.** No exceptions —
   including for your own assertions about tooling and process. Infrastructure
   prose gets trusted exactly as much as a finding.
3. **Prefer source to documentation.** Where they disagree, record both and flag
   it; the disagreement is itself a finding.
4. **Record, do not conclude.** Mechanisms, causes and recommendations are not
   yours. If a conclusion is forming, write it as an `inferred` claim with its
   reasoning, or put it in `unexpected`.
5. **Cheap before expensive**, and when something expensive is needed, say what
   it will cost before starting it.
6. **`unexpected` is not optional.** The thing noticed while chasing something
   else is often worth more than the thing you were sent for.

## 7. Hand over

Emit the records to the map location and tell the user, in two or three lines:
what was surveyed, at what resolution, what is blank, and **what the direction
still cannot answer**.

Then say the map needs compiling — that is `cartographer`'s job, and records
that are never compiled are not a map.
