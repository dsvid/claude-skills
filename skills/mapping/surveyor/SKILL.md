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

### 🛑 Then ask the second question: **how far out does this reach?**

**The direction says what to answer. The reach says how far to go to answer
it** — and they are genuinely separate decisions the user is entitled to make.
**Ask it explicitly and record it beside the direction.** Offer three:

| Reach | What is in scope | Typical ask |
|---|---|---|
| **Inward** | The subject itself — its architecture, structure, internals, how its parts relate | *"map this repo"* |
| **Outward** | The **ecosystem around it** — sibling and satellite repos, the people who build it, the tools, test suites, results corpora and downstream consumers that exist elsewhere | *"what else is out there that bears on this?"* |
| **Both** | The subject **and** its surroundings, at stated resolutions — the common answer, and the expensive one | *"what is this and what surrounds it?"* |

🛑 **A clause inside the direction is not a reach, and does not survive to the
fan-out.** Measured: a run whose direction explicitly asked *"what related repos
exist for testing, validation and profiling"* surveyed the subject thoroughly
and its ecosystem barely at all. **The words were there and nothing acted on
them.** Reach is a separate recorded field because a sentence is not a budget.

**Reach is what authorises spending, and what forbids it.**

- **Inward** ⇒ do **not** spend calls going outward. Note what you see pointing
  out, as `not_determined`, and stop.
- **Outward** ⇒ the enumeration and person-tracing moves in §3 are **obliged,
  not optional**, and the fan-out is wider — price it before dispatching (§3).

⚠ **Ask once, at the start.** ✅ **This is the one breadth question that IS
front-loadable.** *Which* gaps a survey will fail to reach is unknowable now —
that is what §7's gate is for. **Which direction it should travel is knowable
now, and only the user can say.**

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

### 🛑 Read the returned exceptions for **referrals**, and dispatch late

**The fan-out is chosen at the moment of least knowledge.** You pick the
subjects before anyone has read anything; a subject discovered *during* the run
has nowhere to go, because every agent has been told to stay in its lane and
another agent is covering the rest.

**So when an exception refers work to someone — "worth a closer look by whoever
is mapping X" — check that X was actually dispatched.** If nobody owns it,
**you decide: dispatch a late agent, or record it and say so in the hand-over.**
Either is fine. **Silently compiling is not** — the referral was addressed to a
recipient you are responsible for creating.

⚠ **Observed failure:** a sub-agent wrote *"worth a closer look by whichever
agent is mapping the overall testing/validation story."* No such agent existed,
the orchestrator compiled anyway, and the referral pointed at the single
highest-value subject in the whole run.

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

**When the reach is outward, the anti-browse rule above does not bound you — it
bounds what you read *in depth*, never what you *count*.** A "what else is out
there" remit obliges the **whole set**, at **R0** if that is all you can afford.

**The territory does not point at itself.** Measured on a scored run: the
subject's source contained **zero** references to the ecosystem that owned its
test suite, its golden corpus, its tracer **and its benchmark harness**. *None
of it was reachable by following references.* **Outward reach needs its own
moves, and these are the cheap ones — one call each:**

1. **Enumerate the owning organisation or namespace in full.** Not the six you
   happened to hear about. ⚠ A partial enumeration presented against a "what
   exists" remit **reads as complete**, and is the one incompleteness the reader
   cannot detect.
2. 🎯 **Rank the people, then follow them.** Who builds this? What else do they
   build? An ecosystem is usually **one or two individuals' other repositories**,
   and nothing in the subject links to them.
   🛑 **Scope the ranking to what the subject itself added.** On a fork, a
   whole-repo contributor list is the *upstream's* — measured: the ecosystem's
   author ranked **84th** across the tree and **2nd** across the fork's own
   directories. One flag, decisive difference.
   ⚠ **Merge identities before ranking** (the same person appeared twice under
   two emails), and note that a `<login>@users.noreply...` address hands you
   their account name directly.
3. **Resolve forks to their parent.** A vendored fork names the upstream it came
   from, and that is a reference, not a listing.
4. **Then filter the result by the direction** — by what each thing *is*, from
   its description. **This is the point where you stop enumerating and start
   choosing**, and it is the only place browsing is a real risk.

⚠ **Under inward reach, do none of this.** Record what you notice pointing
outward as `not_determined` and leave it. **The user priced this at §0.**

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

`resolved_by` is the deliverable, and a resolver nobody can price is a resolver
nobody runs. **State it as a concrete operation — the actual command, query or
request** — so that reading it tells you what it costs. *"Investigate the
build system"* is not a resolver; *"read `subprojects/foo.wrap` and record the
URL it points at"* is.

🛑 **The band is how many calls and how long. It is never "local vs external",
"code vs web", or "in scope vs out of scope."** A single non-mutating request is
**cheap wherever it lives** — one API call, one fetch, one `git show` all cost
seconds and belong in the same band as a grep.

⚠ **Measured failure:** a run filed *"direct fetch of the org's repository
listing"* in its **most expensive band, 34th of 35 items**, because the resolver
left the machine. It was one API call, roughly two seconds, and the cheapest
unresolved item in the register — and its output held the run's single most
valuable finding. **Nothing downstream could rescue it: the compiler, and the
human, both read the band and believed it.**

**If a resolver turns out to be one cheap call, you are the last honest place to
notice** — see §7.

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

**This is the one gate in the skill, and it is placed here deliberately.** At
§0 nobody — you or the user — knows what the survey will fail to reach; that
knowledge exists only now. **Asking how broad to go at the start cannot fix a
gap that does not exist yet.**

- ✅ **If the user says go, run them and fold the results into the records** as
  ordinary claims with full provenance. **Bounded: one round.** Their results
  authorise no further calls — a survey that resolves its own findings
  recursively has stopped surveying.
- ✅ **If they say hand over as-is, that is a fine answer** and the entries stay
  `not_determined`. The point is that a human priced it, not that it got done.
- 🛑 **Never skip the gate because the list looks small.** The item most likely
  to be on it is the one you mis-filed as expensive.

### Then hand over

Emit the records to the map location and tell the user, in two or three lines:
what was surveyed, at what resolution, what is blank, and **what the direction
still cannot answer**.

Then say the map needs compiling — that is `cartographer`'s job, and records
that are never compiled are not a map.
