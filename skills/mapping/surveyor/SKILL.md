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

Check the existing index for the areas your direction touches. **It tells you two
opposite things, and you need both.**

- **What to skip.** Note each area's `resolution`. **Re-surveying ground already
  covered at sufficient resolution is the single most common waste.**
- 🛑 **What to re-check.** Note each entry's `volatility` and `last_checked`.
  **A `live` claim needs re-surveying *because* it was surveyed before** — an
  open-issue count, a backlog figure, a "currently failing" list. Re-checking a
  volatile claim is not waste; it is the only thing keeping the map honest.

**Build both lists before gathering anything: the skip list and the re-check
list.** A survey that only ever adds new subjects lets the map fill with
confident stale numbers that still carry perfect provenance — which is the
precise defect this whole format exists to prevent.

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

**Group by arc, not strictly by document.** The unit is whatever must be read
*together and in order* to be understood — a set of findings written across one
investigation, a thread and the code it argues about. Splitting an arc forces
every agent to re-read the others for context; that is the most common way a
fan-out costs more than it saves.

### Write the brief to a file

Put the direction, the record schema, the tiers, and the rules in **one file**,
and have every sub-agent read it first. Then each dispatch prompt carries only
the assignment.

**Every sub-agent gets the direction verbatim** — that is what the shared file
guarantees. They start cold, and without the direction they will report what is
interesting rather than what is relevant.

### Choose the model before dispatching

**Sub-agents inherit your model unless the dispatch overrides it**, so an
expensive session silently fans out expensive agents. Most of the sub-agent job
is extraction against a fixed schema, and does not need the model doing the
compiling.

**Ask which model to fan out on, and default to Sonnet.** Ask *here*, not at
§0 — only now do you know the fan-out width, so only now can you price it
("14 subjects across ~40 documents"). Below the fan-out threshold, skip the
question. If the user has said "just go", take the default without a round trip.

**If the chosen model is unavailable, fall back to your own, say so in one line,
and continue.** Never fail a survey over a model choice.

### Bound what comes back

Sub-agents **write their records to disk and return a summary, not the records**.
Returning both spends the corpus twice and clutters exactly the context the
fan-out exists to protect.

Require back, in ~15 lines: where the records were written, counts by status, and
**only the exceptions** — what is contested, what could not be determined, what
was unexpected. Everything else is in the file, and you can read it if you need it.

### Give each one a scope, and mean it

Say **which material is theirs and that the rest belongs to other agents.**
Without that, sub-agents read the whole corpus "for context" — n agents each
reading everything is the worst case, and it is the default behaviour.

They may follow a reference out of their assignment to resolve a specific
question. They may not go and read around.

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
unexpected: <free text — anything noticed that the direction did not ask for.
             🛑 LEADS ONLY. Nothing here is a claim, and the cartographer may
             not promote it to one. If it is falsifiable, it belongs in
             `claims:` with full fields — see rule 4>
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

### 🛑 A derived statistic carries its `n` and its spread, in the statement

Reproducibility is not enough. A median computed over **one** run and a median
computed over **eight** are both perfectly reproducible, both correctly
provenanced, and only one of them means anything.

- **State `n` in the statement itself**, not in the provenance line. A reader
  comparing two numbers must see what each was computed over without leaving the
  sentence.
- **State the spread** (sd, or range) whenever the claim asserts that two values
  **differ**. A difference smaller than the within-group spread is not a
  difference, and you cannot know that without computing it.
- 🛑 **Never describe a subset as its population.** If the figure is run 1, the
  statement says *run 1* — not "the control arm". Naming the subset in one half
  of a comparison and dropping it in the other is the specific error that
  produced this rule.
- If `n` or the spread cannot be established, the claim is `not_determined`, and
  `resolved_by` is the recomputation.

⚠ **Watch the asymmetry that hides this.** A recomputation that *confirms* an
existing figure tends to carry `n`, because a prior claim is there to check
against. A recomputation that *asserts something new* — exactly the one worth
most and checked least — tends not to. **The novel claim needs it more.**

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
   reasoning.
   🛑 **`unexpected` is not the overflow for that.** It takes *leads* — things
   worth someone's attention — never a falsifiable assertion, and never a
   number. **Anything falsifiable goes in `claims:` with grounding, tier,
   provenance, accessed, and (if derived) `n`**, where the schema forces the
   discipline. Writing it as free text is how an unchecked figure reaches a map
   wearing the authority of a checked one.
5. **Cheap before expensive**, and when something expensive is needed, say what
   it will cost before starting it.
6. **`unexpected` is not optional.** The thing noticed while chasing something
   else is often worth more than the thing you were sent for.
7. 🛑 **Write only to the map location. Never edit the domain's own documents or
   source** — surveying must not mutate the territory. You will find stale,
   wrong and superseded material; **that is a record to emit, not a thing to
   fix.** The artefacts you are most tempted to correct are the ones the human
   relies on most, and a survey that quietly rewrites its own subject has
   destroyed the evidence for its own findings.
8. 🛑 **Never run a `git` command that writes.** Read-only git is one of the
   survey's best instruments — `log`, `show`, `blame`, `diff`, `ls-files`,
   `rev-parse`. **`commit`, `add`, `checkout`, `switch`, `branch`, `stash`,
   `merge`, `rebase`, `reset`, `restore`, `clean`, `tag`, `push` are not** — in
   the surveyed repo or any other.
   - **The territory is usually a repo the human does not own**, whose `origin`
     may be a real upstream, and whose working tree may hold uncommitted work or
     a branch that is load-bearing somewhere else.
   - 🛑 **A `checkout` is a mutation.** "Just look at `main`" can discard
     uncommitted state and move a branch the human was in the middle of.
   - 🛑 **Committing map files into the surveyed repo is the specific accident
     to avoid.** It puts your output in someone else's history, one `push` from
     being public and attributed to them.
   - **If the map location would sit inside the surveyed repo, say so and ask
     for somewhere else.** If you need another revision, ask the human for a
     worktree or a second clone — **do not make one by moving theirs.**

## 7. Hand over

Emit the records to the map location and tell the user, in two or three lines:
what was surveyed, at what resolution, what is blank, and **what the direction
still cannot answer**.

Then say the map needs compiling — that is `cartographer`'s job, and records
that are never compiled are not a map.
