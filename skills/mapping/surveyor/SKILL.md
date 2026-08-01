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

### 🛑 Then ask the **reach** — how far out does this go?

**The direction says what to answer; the reach says how far to go to answer
it.** Separate decisions, both the user's. **Ask explicitly, record beside the
direction**, offer three:

| Reach | In scope | Then |
|---|---|---|
| **Inward** | The subject itself — structure, internals, how its parts relate | **Spend nothing going outward.** What points out is a `not_determined` |
| **Outward** | The ecosystem around it — sibling repos, the people who build it, the tooling, test suites and results corpora living elsewhere | §3's enumeration and person-tracing are **obliged, not optional**; fan-out is wider, so price it |
| **Both** | Both, at stated resolutions — the usual answer, and the expensive one | As above |

🛑 **A clause inside the direction is not a reach.** Measured: a run whose
direction explicitly asked *"what related repos exist for testing"* surveyed the
subject thoroughly and its ecosystem barely at all. **The words were in every
sub-agent's brief and nothing acted on them.** A sentence is not a budget.

✅ **This is the one breadth question that is front-loadable.** *Which* gaps a
run will fail to reach is unknowable now — §7's gate handles that. Which
direction it travels is knowable now, and only the user can say.

**Reach is a pass, and passes are additive.** `inward` now, `outward` later is
first-class, and often better: a usable map lands sooner, and the user prices
the ecosystem once they can see the subject.

- **Say which pass this is** — in the hand-over and in the records.
- 🛑 **Without `reach`, a blank ecosystem and an ecosystem checked-and-empty are
  indistinguishable.** One means *go look*, the other *already answered*. §2
  reads this field for exactly that.
- **A later pass adds topics beside the earlier ones and compiles normally. Do
  not re-survey the subject to "re-integrate" it** — that is §2's waste.

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
- 🛑 **What was never in scope.** Read the **reach** of the passes already run
  (§0). **An area missing from an `inward`-only map is unmapped, not empty** —
  and it carries no `resolution` to warn you, because no record was ever
  written about it. **This is the one gap the skip list cannot see**, and on a
  second pass it is usually the whole job.
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

🛑 **And the reach on its own line** (§0) — *"reach: outward. Enumerating the
namespace and tracing contributors is in scope and expected."* **A sub-agent
told only its subject surveys only its subject**, and a reach buried inside the
direction is the clause §0 measured going unread.

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

### 🛑 Read the returned exceptions for **referrals**, and dispatch late

**The fan-out is chosen at the moment of least knowledge**, so a subject
discovered *during* the run has nowhere to go — every agent was told to stay in
its lane.

**When an exception refers work on — *"worth a closer look by whoever is mapping
X"* — check that X was dispatched.** If nobody owns it: **dispatch a late agent,
or say so in the hand-over.** Either is fine; **silently compiling is not.**

⚠ **Observed:** a referral to *"whichever agent is mapping the testing story"* —
no such agent existed, the orchestrator compiled anyway, and it pointed at the
run's single highest-value subject.

### Give each one a scope, and mean it

Say **which material is theirs and that the rest belongs to other agents.**
Without that, sub-agents read the whole corpus "for context" — n agents each
reading everything is the worst case, and it is the default behaviour.

They may follow a reference out of their assignment to resolve a specific
question. They may not go and read around.

**Follow references; do not browse.** Survey what the material actually points
at — what the code imports, what the docs cite, what the issues link. A
plausible-sounding name in a listing is not a reference.

### 🛑 Outward reach (§0): enumerate completely, then read selectively

**Outward reach suspends the rule above for *counting*, never for *reading in
depth*.** It obliges the **whole set**, at **R0** if that is all you can afford.

🛑 **The territory does not point at itself, so following references cannot get
you there.** Measured: a subject's source held **zero** references to the
ecosystem owning its test suite, golden corpus, tracer and benchmark harness.
**Four moves, one call each:**

1. **Enumerate the owning namespace in full** — not the few you happened to hear
   about. ⚠ A partial enumeration **reads as complete**, and is the one
   incompleteness a reader cannot detect.
2. 🎯 **Rank the people, then follow them.** An ecosystem is usually one or two
   individuals' *other* repositories, and nothing in the subject links to them.
   🛑 **Rank over what the subject itself added** — on a fork the whole-repo
   list is the upstream's: measured, the ecosystem's author ranked **84th**
   across the tree and **2nd** across the fork's own directories. ⚠ Merge
   identities first (one person, two emails); `<login>@users.noreply…` gives
   you the account name.
3. **Resolve forks to their `parent`** — that is a reference, not a listing.
4. **Then filter by the direction**, on what each thing *is*. **Stop enumerating,
   start choosing** — the only point where browsing is a real risk.

⚠ **Under inward reach, none of this.** It is a `not_determined`; the user
priced it at §0.

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
reach: inward | outward | both   # §0 — recorded so a later reader knows
                                 # whether a blank ecosystem was a finding
                                 # or simply out of scope
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

### 🛑 Price a `resolved_by` by the operation, never by its category

A resolver nobody can price is a resolver nobody runs. **State it as a concrete
operation — the actual command, query or request** — so reading it tells you
what it costs. *"Investigate the build system"* is not a resolver; *"read
`subprojects/foo.wrap` and record the URL"* is.

🛑 **The band is how many calls and how long — never "local vs external" or
"in scope vs out".** One non-mutating request is **cheap wherever it lives**:
an API call, a fetch and a `git show` all belong in the same band as a grep.

⚠ **Measured:** a run filed *"direct fetch of the org's repository listing"* in
its **most expensive band, 34th of 35**, purely because it left the machine. One
call, ~2 seconds, the cheapest item in the register — and its output held the
run's most valuable finding. **The compiler and the human both read the band and
believed it.** If a resolver is one cheap call, **you are the last honest place
to notice** (§7).

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
8. 🛑 **The subject is a READ-ONLY ARTEFACT. Observe it; never change it.**
   **This is a property of the subject, not a list of banned commands** — if an
   action alters bytes in the territory, it is out of scope, whether or not it
   appears below. Treat every subject as someone else's, because it usually is.
   - **Read-only git is one of your best instruments** — `log`, `show`, `blame`,
     `diff`, `ls-files`, `rev-parse`. Use it freely.
   - **Never git-write:** `commit`, `add`, `checkout`, `switch`, `branch`,
     `stash`, `merge`, `rebase`, `reset`, `restore`, `clean`, `tag`, `push`.
     🛑 **A `checkout` is a mutation** — "just look at `main`" can discard
     uncommitted work and move a branch the human was in the middle of.
   - **Never write by any other route either:** creating files inside the
     subject, running builds, installs, formatters or codegen that write into
     its tree, deleting anything, or "fixing" the stale README you just found
     wrong. **You will find wrong, dead and superseded material — that is a
     record to emit, never a thing to repair** (rule 7).
   - 🛑 **Nothing may leave the machine.** No PRs, no issues, no comments on the
     subject's tracker, no publishing — **even when what you found is a real bug
     in it.** Emit a record and hand it to the human.
   - **The map location is the only place you write.** If it would sit inside
     the subject, stop and ask for somewhere else. If you need a different
     revision, ask for a worktree or a second clone — **do not make one by
     moving theirs.**
   - **Why absolute:** the subject's `origin` may be a real public upstream, and
     a commit puts your output in someone else's history one `push` from being
     public and attributed to them. **The cost of asking is a sentence; the cost
     of being wrong is not recoverable by you.**

## 7. Hand over

### 🛑 First: the cheap-band gate. Show what one more step buys, and ask

**Before handing over, re-read your own `not_determined` entries and pull out
every resolver that is a single cheap, non-mutating operation** (§5). Then stop
and put them to the user, with what each would buy:

> *3 open questions resolve in one call each: enumerate the org's repos
> (`gh api orgs/<org>/repos`); read the dependency manifest for the URL it
> points at; grep the remaining CI workflows for test steps. ~2 minutes total.
> Run them, or hand over as-is?*

**Placed here deliberately** — §0 settles which *direction* to travel; only now
does anyone know what the run actually failed to reach.

- ✅ **Go ⇒ run them and fold the results in** as ordinary claims with full
  provenance. **Bounded: one round.** Their results authorise no further calls —
  a survey that resolves recursively has stopped surveying.
- ✅ **Hand over as-is is a fine answer**; the entries stay `not_determined`.
  The point is that a human priced it, not that it got done.
- 🛑 **Never skip the gate for being small.** The likeliest item on it is the one
  you mis-filed as expensive.

### Then hand over

Emit the records to the map location and tell the user, in two or three lines:
what was surveyed, at what resolution, what is blank, and **what the direction
still cannot answer**.

Then say the map needs compiling — that is `cartographer`'s job, and records
that are never compiled are not a map.
