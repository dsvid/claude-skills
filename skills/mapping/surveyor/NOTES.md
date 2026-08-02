# surveyor — notes

Append friction as it happens: what was ambiguous, what had to be worked around,
what the human corrected. One line each. This is the evidence for promoting the
maturity rung.

## Open questions, written before the first run

Pre-registered so a first run can answer them rather than just feel fine.

- **Does refusing to start without a direction help or obstruct?** It is the
  stopping rule and the fan-out instruction, so it looks load-bearing — but a
  user who "just wants a look around" may find it pedantic. Watch whether the
  direction that gets written is real or invented to satisfy the gate.
- **Is the YAML record too heavy to actually fill in?** The likeliest failure of
  this skill. If records come back thin, half-filled, or with `provenance` doing
  the work of four fields, the schema is wrong, not the surveyor.
- **Will the tiers get used, or will everything be recorded as T2?** T0 and T3
  in particular. If nothing is ever T0, that probably means unprovenanced claims
  are being quietly dropped or dressed up rather than marked.
- **What is the sub-agent fan-out threshold?** "More than a couple of subjects"
  is a guess. Below some N the coordination costs more than it saves.
- **Does `unexpected` get filled?** It exists because the by-product often beats
  the target. An empty `unexpected` on every record means fan-out is flattening
  exactly what it was meant to preserve.
- **Does step 2 actually fire?** "Ask what existing material can still answer
  this" and "has someone already answered this" are the two cheapest steps and
  the two most likely to be skipped in favour of gathering.

## Friction log

<!-- newest first; date each entry -->

### 2026-07-31 — first dogfood run (the-xemu-cartographer, direction: PGR1 #480)

**🛑 Cost. 5 sub-agents, ~528k sub-agent tokens, ~28 min wall.** The human's
call, and it is right: *"probably it's something that should use a lower model
like Sonnet and not a higher model like Opus."*

**Sub-agents inherit the parent's model unless the dispatch overrides it**, so an
Opus session silently fans out five Opus agents. The §3 work is **bulk extraction
against a fixed schema** — read a document, classify each claim live/superseded/
refuted/contested, fill fields. That is not the parent's job (compiling,
adjudicating, deciding what to fan out to) and it does not need the parent's
model.

⇒ ✅ **DONE same day. `SKILL.md` §3 now asks which model to fan out on, defaults
to Sonnet, and falls back to the parent's model if the choice is unavailable.**
⚠ Untested whether Sonnet holds the supersession discipline, which is the one
part of the sub-agent job that is *judgement*, not extraction. Worth one A/B on
a document with a known-dead claim in it before committing the change.

**What worked, unprompted and worth keeping:**
- **The supersession framing carried.** Every sub-agent returned dead/contested
  claims as *findings* rather than tidying them away. `contested` was used, and
  used correctly — two documents disagreeing with neither withdrawing.
- **`unexpected` earned its place, twice.** Two harness defects
  (`analyse-results.py` crashes on the newest data; `scenes.psd1` is 0 bytes) and
  a corpus-level structural observation. Neither was asked for; both are real.
- **One sub-agent independently recomputed the headline numbers from raw CSVs**
  rather than trusting the write-up. Nothing in the skill asks for that. It
  probably should.
- **§2's gate fired for real** — the direction was fully answerable from material
  already in hand, no new gathering, no web, no execution.

**Friction:**
- **The R0-R3 ladder barely bit.** Every record came back R2-R3, because a
  document corpus you can read exhaustively is R3 almost by construction. The
  ladder is built for territory you *sample*. Not wrong, but it did no work here.
- **Records-to-report ratio is poor.** 4,424 lines of YAML; the useful artefact
  was the ~150-line hand-written report. The YAML is the cartographer's input, so
  this may be fine — **but it is untested, because the cartographer has not run
  on it yet.** If the compile does not need that much, the schema is too heavy.
- ✅ **FIXED same day, §3.** ~~Assigning by document, not by subject, was the
  right call and the skill does not say so.~~ §3 said "one sub-agent per
  subject". The natural unit was "one per *cluster of documents written in one
  arc*" — F015/F016/F017 only make sense read together and in order. **§3 now
  says "group by arc, not strictly by document".**
- ✅ **FIXED same day, §3.** ~~Scope discipline needed to be added by hand.~~
  Sub-agents will read the whole corpus unless told not to; I had to write "read
  only your assigned documents, others have the rest" into the brief. **§3 now
  carries it, with the reason: n agents each reading everything is the worst
  case, and it is the default behaviour.**
- ✅ **FIXED same day, §3.** ~~The shared-brief-in-a-file pattern worked well and
  is not in the skill.~~ Write direction + schema + rules to one file, have every
  sub-agent read it first. Keeps dispatch prompts short and guarantees the
  direction is verbatim. **§3 now has a "Write the brief to a file" subsection.**

**Against the pre-registered success criterion** (`sessions/009`: *success =
compiling surfaces a correction, or a `contested` entry we had been treating as
settled*):
⚠ **Surveying alone already surfaced 11 traps and 6 contested claims** — before
the cartographer ran. **That is a problem for the skill boundary, not a win**:
if the surveyor's own output answers the question, s009's kill criterion
("cartography is a phase of the surveyor, not a skill") is live and pointed at
`cartographer`, not at this one. **Do not score this run until the compile has
run and shown it adds something the report does not.**

---

# Pending changes — sub-agent cost

**Raised by the human, 2026-07-31, after the first dogfood run.**

> ✅ **PARTLY IMPLEMENTED, same day.** `SKILL.md` §3 now carries: the model
> question (default Sonnet, with the unavailability fallback), the return-payload
> cap (§3a), the scope-discipline instruction, group-by-arc, and the
> shared-brief-in-a-file pattern.
>
> **Still open below:** the Sonnet A/B that gates the *default* (§1); schema
> slimming (§3b, §3c) which waits on `cartographer`; breadth-before-depth
> adherence (§3d); the shared citation index (§3e); read budgets (§3g).
> **None of the implemented items has been tested** — they land in the next run.

> *"Can the skill choose models for the sub-agents? If so we should get it to ask
> the user what model they want to use for the sub-agents and default fallback to
> Sonnet. If not then it should prompt the user to change to whatever model is
> appropriate before doing the sub-agent fan-out. Also if there are any other ways
> to make the sub-agents more token efficient then we should explore that as
> well."*

## 1. Yes, the skill can choose — so the "prompt the user to switch" fallback is moot

**The `Agent` tool takes a per-dispatch `model` parameter** (enum: `sonnet` /
`opus` / `haiku` / `fable`). Per its schema it *"takes precedence over the agent
definition's model frontmatter"* and over inheritance from the parent. So the
fan-out can run on Sonnet from an Opus session with no user action and no session
switch. `[source: Agent tool schema, this session, 2026-07-31]`

Two caveats worth carrying into the edit:
- **It is ignored for `subagent_type: "fork"`** — forks always inherit the parent
  model. §3 fan-out does not use forks, so this does not bite, but the skill
  should not *say* "always honoured".
- **Availability is not guaranteed.** During this very session a Sonnet-backed
  call returned *"claude-sonnet-5 is temporarily unavailable"*. **The skill must
  not hard-fail the survey when the chosen model is down** — fall back to the
  parent's model, say so in one line, and continue.

⇒ ✅ **IMPLEMENTED same day, per the human's shape: ask which model, default
Sonnet.** All four bullets below are now in `SKILL.md` §3 ("Choose the model
before dispatching"). Kept here as the reasoning behind the wording:
- Ask **once per survey, at §3 dispatch time — not at §0**. At §0 the user has no
  idea how many subjects there will be, so the question is unanswerable. By §3
  the skill knows the fan-out width and can price it: *"12 subjects, ~5 documents
  each — Sonnet (default) / Opus / Haiku?"*
- **Skip the question entirely below the fan-out threshold.** One or two
  sub-agents is not worth a prompt.
- **Default is Sonnet, and the default should be silent-able.** A user who says
  "just go" should get Sonnet without a round trip.
- ⚠ **Open, and the reason this is not just a one-line edit: is Sonnet good
  enough at the part that is judgement?** The extraction is mechanical; deciding
  *live / superseded / refuted / contested* is not, and that classification is the
  entire value of a survey record. **Do one A/B on a document with a known-dead
  claim in it (this repo has four such regression cases in `cartographer/NOTES.md`)
  before committing the default.** If Sonnet tidies dead claims away instead of
  flagging them, the cheap fan-out costs more than it saves.

## 2. Measured cost of the run that prompted this

| Sub-agent | Tokens | Tool uses | Note |
|---|---|---|---|
| `status-data` | 151,381 | 35 | Recomputed Block E from raw CSVs — expensive **and worth it** |
| `f010` | 144,773 | 12 | 3,061-line document, read front to back |
| `f009-plans` | 98,310 | 16 | |
| `f012-f014` | 69,719 | 12 | |
| `f015-f017` | 64,006 | 6 | Cheapest, and returned the cleanest records |
| **Total** | **528,189** | **81** | ~28 min wall |

**Do not optimise this blindly.** The single most expensive agent produced the
independent recomputation of the headline figures, which is among the most
valuable things the run did. The target is waste, not cost.

## 3. Token-efficiency candidates, roughly best-first

**a. Cap the return payload. Biggest win, smallest change.** ✅ **IMPLEMENTED
same day — `SKILL.md` §3 "Bound what comes back".**
Each sub-agent wrote full records to disk **and** returned a long prose summary
to the parent — the same content twice, and the parent does not need it. The
records are the deliverable; the parent needs a pointer plus the exceptions.
⇒ *"Return at most 15 lines: where you wrote the records, counts by status, and
only the dead / contested / newly-doubtful items. Everything else is in the file."*

**b. Stop repeating the direction verbatim in every record.**
The schema puts `direction:` in each record. Across 26 records that is pure
duplication. ⇒ Direction once in a manifest; records reference it. **Check with
`cartographer` first** — if the compile reads records individually, the
redundancy may be load-bearing.

**c. Asymmetric detail: spend tokens on the contested, not the settled.**
A live, uncontested, single-source claim needs one line. A superseded or
contested one needs the full chain. The current schema demands the same weight
for both, which is why 140 claims became 4,424 lines of YAML.

**d. Actually honour §4 (breadth before depth) in the fan-out.**
This run dispatched **R3 on everything simultaneously** — which contradicts the
skill's own ladder. A cheap R0/R1 sweep first (Haiku, or `Explore`, which reads
excerpts rather than whole files) would locate where the supersessions cluster;
only those get an expensive R2/R3 pass. **The skill already says this and the
run ignored it. That is a skill-adherence failure, not a missing feature** — and
it is worth asking why the instruction did not fire.

**e. Deduplicate cross-checking.**
Several sub-agents independently grepped the rest of the corpus to check whether
a claim was cited elsewhere. That is one shared index the parent should build
**once**, before dispatch, and hand to everyone.

**f. Assign by arc, not by document — and say so in §3.** ✅ **IMPLEMENTED same
day.** Also a cost item, not only a quality one: splitting F015/F016/F017 across
three agents would have forced each to re-read the others for context.

**g. Give sub-agents a read budget and an explicit no-re-read rule.**
35 tool uses on one agent suggests re-reading. Untested.

## 4. 🛑 The `status:` field — the one deliberate non-change

**The v1 claim schema has no way to say a claim is dead.** It carries `grounding`,
`tier`, `accessed`, `volatility` — all of which a *superseded* claim satisfies
perfectly. That is the exact failure the whole exercise exists to catch: correct
provenance, correct tag, dead claim.

For the dogfood run I added a field by hand, in the brief, not the skill:

```yaml
status: live | superseded by <ref> | refuted by <ref> | contested
```

**It was the single most valuable field in the run** — every trap and every
contested entry came back through it.

**Not promoted into `SKILL.md`, on purpose.** It is a v1 format change, and the
format is a contract with `cartographer`. And there is a real design question
underneath, which the dogfood cannot answer because the compile has not run:

> **Is currency the surveyor's output or the cartographer's?** If the surveyor
> stamps `status`, the cartographer's index job shrinks to aggregation — possibly
> to nothing, which is s009's kill criterion firing. If the cartographer derives
> status by comparing records, the surveyor should stay dumb and just record what
> each document *says*, including that a document announces its own supersession.

⚠ **Decide this before the next survey, because it determines whether
`cartographer` is a skill.** Do not quietly add the field.

## 5. What to do next

**§1 and §3a/e/f are done** — the model question, the return cap, the scope
instruction, group-by-arc, the shared brief. They were done ahead of the A/B
because the human asked for the ask-with-Sonnet-default shape directly, and
because none of them changes *what a survey finds*, only what it costs.

**So the A/B is now a check on a shipped default, not a gate on an unwritten
one.** That is a weaker position than gating it would have been — if Sonnet
tidies dead claims away, the skill is already telling people to use it.
⇒ **Run the A/B before the next real survey, not after.** A document with a
known-dead claim in it; `cartographer/NOTES.md` holds four such regression cases.

Then, in order:
1. **The `status:` field decision (§4)** — it determines whether `cartographer`
   is a skill at all, so it outranks every efficiency item.
2. **Run `cartographer` on the s010 records.** §3b and §3c (schema slimming) are
   blocked on it: only the compile can say what the records need to carry.
3. §3d (breadth-before-depth adherence), §3e (shared citation index), §3g (read
   budgets) — all untested guesses, cheapest to settle by watching one more run.

### 2026-07-31 — pre-cartographer-run: records carry no claim ids

**The index (`cartographer` §4) points at `claim-id`s. This skill's record schema
emits none** — claims are anonymous bullets under `claims:`. Either the surveyor
starts emitting stable ids, or the cartographer mints them at ingest and says so.
**Currently neither is written down.** Full note, with two other v1 contract
holes and what to watch on run 1: `../cartographer/NOTES.md`.

### 2026-07-31 — §2 only knew how to skip, not how to re-check (raised by the human)

**The human asked whether the index-reading design means future surveys "only
surface new things."** Half yes — and the missing half was a hole.

§2 said only: note the `resolution`, don't re-survey what is covered. **That is
the skip list.** But the record schema carries `volatility: static | slow | live`
and the index carries `last_checked`, and **nothing told the surveyor to use
them.** A `live` claim — an open-issue count, a backlog figure — needs
re-surveying *precisely because* it was surveyed before.

⇒ ✅ **FIXED same day. §2 now says the index tells you two opposite things, and
requires both a skip list and a re-check list before gathering.**

**The loop was broken at the other end too, and this is the part worth
remembering:** `visualiser` §6 already reports *"`live` claims whose
`last_checked` is old, because a diagram makes a stale number look freshly
true."* So the visualiser could *detect* staleness while the surveyor — **the
only one of the three that can actually go and re-check** — had no instruction to
act on it. **A three-skill loop where the detector and the fixer never
connect.**

⚠ **Fixed rather than observed, deliberately, and the rule is now explicit:
defer-to-observe only when run 1 can actually observe it.** Run 1 is a single
compile with no prior index and no second survey, so nothing here was testable —
deferring would have been pure delay. Contrast the claim-id gap in
`../cartographer/NOTES.md`, which run 1 *will* exercise.

---

## The blind xemu run, scored 2026-08-01 — it stops at the edge of gaps it priced

Full scoring: `../SCORE-xemu-architecture-2026-08-01.md`. Headline:
**4 HIT / 17 MISS / 0 WRONG** of 21 oracle rows, **failing 3 of 4 T-must rows**
while asserting nothing false.

🎯 **The single most actionable pattern: every T-must miss was one cheap hop
short, and in two of three cases the surveyor had already written down the hop.**

| Missed row | What it had | The hop it did not take |
|---|---|---|
| **A1** — which thread renders | PFIFO's `QemuThread` from the struct **declaration** in `nv2a_int.h` | Open the thread's **body**. `pfifo.c:452` names `pfifo_thread`; `:464` calls `pgraph_process_pending` |
| **C1** — the `NV2A_PROF_*` counters | `profile.c` (the FPS/mspf struct) | Open the sibling `debug.h`, which holds **53** counters incl. `SURF_TO_TEX_FALLBACK` |
| **D1** — `nxdk_pgraph_tests` | An org-repo enumeration it recorded as **incomplete**, with `resolved_by: "Direct fetch of the org repositories page"` | Make that call. **One `gh api`.** Its output contains `nxdk_pgraph_tests` |

⇒ **Proposed change (NOT yet made — this one is observable, so it is deferred to
run 2 per the rule above): execute a `not_determined`'s own `resolved_by` when it
is a single cheap, non-mutating call.** The surveyor already computes the
resolver and the cost band; it then declines to spend the minute. ⚠ The obvious
failure mode of the fix is unbounded fan-out — the bound has to be *one*
non-mutating call, not "resolve what you can."

🛑 **Not just a `not_determined` problem — C1 and A1 were never even asked.** A
weaker, always-on version that would have caught both: **when a claim rests on a
declaration, read the definition; when it rests on one file in a directory,
list the directory.** Neither is fan-out; both are the same file's neighbours.

✅ **What the run vindicates, and it is the load-bearing half:**
- **Zero fabrication across 21 keyed rows and 12 records.** Unknown areas were
  drawn blank, marked **Thin** in the index, and routed to `OPEN_QUESTIONS.md`.
- **It produced a negative** — "xemu's CI runs no tests" — corroborated two
  independent ways and *promoted* to Corrections owed. The oracle called
  negatives the hardest thing for a survey to produce; this is the evidence it
  can.
- **Its own hedging was calibrated.** The entry it labelled *"most likely to be
  wrong if re-verified"* turned out **right**, and was the run's most valuable
  find (`the-xemu-cartographer/findings/F019`).

⚠ **Precision failure mode seen here is counting, not claiming:** "14 workflow
files" (there are 15) and "12 org repos" (13). Both are derived integers from a
listing, both trivially recomputable, neither recomputed. Same family as the
`n`-rule in `../cartographer/NOTES.md`.

### ⚠ Supersedes the "one hop short" framing above — the disease is the **cost model**

The section above says the run "declines to spend the minute". **That is the
symptom, not the cause, and the human found the cause by asking whether the run
had flagged its own gap.** It had — and more usefully than "flagged":

`map_fresh/OPEN_QUESTIONS.md` entry **34 of 35**, under the heading
**"Expensive (requires new survey scope, external fetch, or org-wide
enumeration)"**:

> *Full enumeration of the remaining repos in the xemu-project org (search
> reported 12 total; only 6 individually named) — resolved by: direct fetch of
> `https://github.com/orgs/xemu-project/repositories`*

**Second-to-last item in the register, bottom cost band. It costs one `gh api`
call — about two seconds.** It is simultaneously the cheapest unresolved item in
the whole register and the most valuable: its output contains
`nxdk_pgraph_tests` (T-must row D1) and `xemu-test` (→ `F019`).

🛑 **The cost bands sorted by category, not by the call.** Local grep ⇒ cheap;
anything leaving the machine ⇒ expensive. **"External fetch" is not a cost.**
One API call and a week of crawling landed in the same band, so nothing
downstream — sub-agent, orchestrator, compiler, human — ever saw it as
actionable.

⇒ **Revised fix, replacing the "execute a one-call resolver" proposal above,
which treats the symptom:**

1. **Price a `resolved_by` by the operation, not its category.** The band is
   *how many calls and how long*, and a single non-mutating request is **cheap
   wherever it lives**. A `resolved_by` that cannot be priced this way is
   itself a signal the resolver is vague.
2. **Gate before the compile, and ask.** Show the human the top of the cheap
   band with what each would buy, and take a go/no-go. ⚠ **The trigger point is
   the end of the survey, not the start of the run** — see below.

🛑 **Why "just front-load how broad to go" does not fix this, though it is the
right instinct.** At t=0 nobody — human or agent — knows the xemu-project org
holds a test harness. **That knowledge only exists mid-run**, in one sub-agent,
which correctly wrote it down and handed it off. Its `unexpected` field even
names the recipient: *"worth a closer look by whichever agent is mapping xemu's
overall testing/validation story."* **No such agent existed.** The handoff was
addressed to a subject nobody had been assigned.

⇒ **Two distinct failures, and only one is a pricing bug:**
- **The register was mispriced** → fix 1 above.
- **A sub-agent's referral had no addressee.** The fan-out is fixed at dispatch,
  so a subject discovered *during* the run has nowhere to go. ⇒ **The
  orchestrator must read the returned exceptions for referrals and decide
  whether to dispatch a late agent — before compiling.** Currently §3's
  "bound what comes back" collects exceptions and nothing acts on them.

✅ **What IS front-loadable, and would have worked:** the direction itself —
*"what related repos exist for testing, validation and performance profiling"* —
**mandates an enumeration**, and that is knowable at t=0 from the direction
alone. See the anti-browse carve-out proposed in the scoring doc.

### ✅ APPLIED 2026-08-01 — four edits, and what run 2 must show

The three fixes above are now in the specs. **The pre-registration for the next
run is here, written before it happens.**

| Edit | Where | Predicted effect |
|---|---|---|
| **Anti-browse carve-out** — a "what exists" direction obliges a complete enumeration, at R0 if need be | `surveyor` §3 | The org listing gets counted. **Falsified if run 2 again reports a partial org enumeration as if complete** |
| **Referral check + late dispatch** — an exception addressed to an agent nobody dispatched is the orchestrator's to resolve | `surveyor` §3 | A discovered subject gets an owner, or gets named in the hand-over |
| **Price by the operation, not the category** | `surveyor` §5 + `cartographer` §5b | One-call resolvers land in the cheap band. **Falsified if anything resolvable by a single call lands below the cheap tier again** |
| **The cheap-band gate — show one-call resolvers and ask** | `surveyor` §7 | The human prices it, once, with the list in front of them |

🛑 **Confound to respect when scoring run 2.** These edits were written by an
agent that had just read the answer key, and they name the xemu org enumeration
**explicitly, as a worked example**. A run against xemu therefore cannot show
the rules generalise — it can only show they fire at all. **The stronger test is
run 2 on a different subject**, where nothing in the spec points at the answer.
Same defect as `EVALUATION.md` row 2 (*"a rule that fires when the agent knows
it is being marked is not shown to fire otherwise"*), and it is now twice in a
row — worth treating as a standing property of self-scored runs, not an
incident.

⚠ **Cost of the edits: `surveyor/SKILL.md` 291 → ~360 lines.** Four rules added
to a spec read in full on every run. If the next scoring shows no movement,
**cut them rather than adding a fifth.**

⚠ **Not fixed, deliberately:** the declaration-vs-definition gap (oracle rows
A1, A3, A5 — read the struct, never the thread body or the lock sites). It is
3 rows, and every phrasing reduces to *"read more of the code"*, which is not a
rule. **Left open.** If run 2 misses those rows again with the four fixes in
place, that is the evidence that it needs its own mechanism.

### 🎯 Proposal (human's, 2026-08-01, MEASURED): reach the ecosystem via the **contributor graph**

**The question:** when a subject's ecosystem lives outside it — other people's
repos, with no reference from the code — is it findable at all without knowing
what to look for?

**On the xemu run, the answer from source alone is no.**
`git grep -i "abaire\|nxdk_pgraph\|xbdm"` across the whole subject at the
surveyed SHA: **zero matches.** The ecosystem that owns its test suite, its
golden corpus, its tracer *and its benchmark harness* is **invisible from the
code**.

**The human proposed: go to key contributors, then to what else they build.**
Measured, and it works — **but the naive form fails badly, and the failure is
the interesting part.**

| Ranking method | Where the ecosystem's author lands |
|---|---|
| `contributors` API, whole repo | 🛑 **rank 84** |
| `git shortlog` scoped to the paths **the fork itself added** | ✅ **rank 2** |

🛑 **Why the whole-repo ranking is worthless on a fork:** the subject is a fork
of a much larger upstream, so its history carries the upstream's contributors.
**The top 14 were upstream maintainers with no involvement in the fork's own
work.** The fork's author was 15th; the ecosystem's author 84th.

✅ **The fix is one flag: rank contributions over the fork-specific paths.**

```
git shortlog -sne <sha> -- <the dirs the fork added>
→ 763 <fork author> · 107 <ecosystem author> · 43 · 24 <same person, 2nd email> · 18 …
```

⚠ **Two practical caveats, both hit in the measurement:**
- **One person, two identities.** The #2 contributor also appears at #4 under a
  second email. **Naive counting splits them** — merge by name before ranking.
- **`<login>@users.noreply.github.com` hands you the account name directly.**
  That is the hop from a commit to a person's other work.

⇒ **The chain, general and cheap:** scope the log to what the subject actually
added → rank authors → resolve to accounts → enumerate their repos → **filter by
the direction**, never by browsing.

🎯 **Why this beats the fork-`parent` route** (the other proposal on the table):
`parent` only fires if the organisation happened to fork the repo — an accident
of how they vendored it. **The contributor graph exists for every subject under
version control**, and it is the same one API call.

⚠ **NOT APPLIED. Deliberately.** Four rules went into `SKILL.md` this session
(291 → 372 lines) and this file already says: **if the next scoring shows no
movement, cut them rather than adding a fifth.** ⇒ **If a fifth is added, it
should be this one and not the `parent` rule** — same cost, strictly wider
coverage. Decide after run 3, not before.

### ✅ APPLIED — **reach** as a first-class §0 question (human's design, 2026-08-01)

*"Where it goes out should depend on the remit given upfront. It should ask me
if I want to map this repo's architecture, or if identifying the ecosystem of
supporting repos is part of the survey."*

⇒ **§0 now asks two questions, not one: the direction, and the reach**
(`inward` / `outward` / `both`), recorded as a schema field.

🎯 **This resolves three open problems at once, which is why it earned a §0 slot
rather than a fifth rule:**

1. **It corrects a claim made earlier in this file.** I argued breadth cannot be
   front-loaded because the gaps do not exist yet at t=0. **True of *which*
   gaps; false of the *axis*.** Inward-vs-outward is knowable at the start and
   is the user's to decide. §7's gate still handles the unknowable half — the
   two are complementary, not competing.
2. **It gives the anti-browse carve-out a principled trigger.** The carve-out
   previously keyed off *parsing the direction's wording* for "what exists",
   which is brittle. **It now keys off a recorded scope decision.** Net line
   cost near zero — it replaced the carve-out rather than adding to it.
3. 🎯 **It converts the contributor-graph proposal from a fifth rule into the
   *content* of a scope the user chose.** The standing note — *cut before adding
   a fifth* — is respected: under `inward` these moves are forbidden, so they
   cost nothing; under `outward` they are the substance of what was asked for.

🛑 **The evidence that a prose clause is not enough:** the scored run's direction
*explicitly* said *"and what related repos, workflows, or tooling exist for
testing, validation, and performance profiling."* It surveyed the subject at R2
across 12 records and its ecosystem at **R1, from web-search snapshots, in one
record**. **The words were in the brief, verbatim, in every sub-agent's file,
and nothing acted on them.** A sentence is not a budget; a recorded reach is.

⚠ **What to watch on run 3:** `both` is the honest answer to most directions and
is also the expensive one. **If every run answers `both`, the field has bought
nothing** — the test is whether outward reach changes what the fan-out actually
dispatches, not whether it changes what the record says.

### Compression pass, 2026-08-01 — before run 3, not after

**Reversed the earlier "consolidate after run 3" note, but only for one half of
it.** The distinction that was missing:

- ✅ **Compression that preserves every rule** — cut repetition, shorten
  evidence, tighten prose. **Does not change what fires, and reduces the
  spec-bloat confound.** Safe before a run, and done now.
- 🛑 **Consolidation that merges or drops rules** — changes the thing under
  test. **Still deferred to after run 3**, when per-rule observation says which
  ones fired.

**Result: 458 → 419 lines. The six sections added today went ~150 → ~110.**
The main win was real repetition: *"an outward clause in the direction did not
survive to the fan-out"* had been argued in **three** places, and is now argued
once at §0 with the other two pointing back.

⚠ **Be honest about the size of the win: ~8%.** The file is still **44% larger
than this morning**, and the remaining bulk (§5 schema, §6 rules) is
pre-existing and not obviously cuttable. **Compression alone will not undo the
growth — only merging or dropping rules will, and that needs run 3's data.**

✅ **Verified after compressing: every rule still present**, by grepping one
phrase per fix, and no domain names leaked back in.

---

## Run 3, 2026-08-01 — the gate fired, and it read the wrong half of the file

Full score: `../SCORE-xemu-architecture-2026-08-01-run3.md`. **6 HIT / 15 MISS /
0 WRONG-on-key** vs run 2's 4/17/0. Two notes belong here; the rest is scoring.

### ✅ The cheap-band gate works. This is the first fix shown firing unprompted.

It stopped before hand-over, priced 11 one-call resolvers at ~2 min total, and
asked. The human said run them → `record-gate-resolutions.yml`, 10 claims,
**4 index entries superseded**. Nothing about the gate needs changing.

### 🛑 …but it only reads `not_determined:`. `unexpected:` leads are invisible to it.

The same run's `open-questions.md` listed **six** free-text leads, each with a
concrete one-call resolver, in a section headed *"Further leads"*. **None was
dispatched.** One of them — `abaire/xemu-dev_pgraph_test_results` — turned out
to be a live per-PR accuracy workflow used on 11 real PRs, and two `gh api`
calls away. It was recovered by the scorer, not the run.

⇒ **Fix: the §7 cheap-band gate must sweep `unexpected:` too.** The map-format
rule *"free text may at most become a lead, never a claim"* is correct and
should stay — but "never a claim" was silently read as "never worth resolving".
**A lead with a one-call resolver is the cheapest claim in the register, not a
non-claim.**

### 🛑 It enumerated the org and *sampled* the person — and the sample dropped two keyed rows.

`gh api users/abaire/repos --paginate` was run and cited **four times**. Two
records (`abaire-16`, `-17`) then described a *selection* from that listing by
description. Dropped, silently: `xemu-nxdk_pgraph_tests_results` (the hardware
golden corpus, a **T-must** row) and `nxdk_ntrc_dyndxt` (T-should).

⚠ **This is the enumeration rule satisfied in form and defeated in substance.**
Run 2 was "one hop short — it had written the hop down". **Run 3 is zero hops
short: the answer was in output it had already fetched.**

⇒ **Fix: a record that filters an enumeration must state the filter and the
count it dropped** — `"116 repos listed, 17 recorded, filtered by description
for xemu/nv2a relevance"`. Then the omission is visible and priceable instead of
invisible. **Enumerating completely and recording selectively are different
acts and the schema currently cannot tell them apart.**

### ⚠ Counting is still not fixed — it moved.

Run 2: 14-vs-15 workflows, 12-vs-13 org repos. Run 3 got **both of those
right** and produced a new one: `subprojects/*.wrap` **35, actually 36**. Worse,
`hwxbox-3` flattened an `if/else` on a machine property into *"registers 5 SMBus
peripherals"* — **false in all three configurations** (max 3).

⇒ The miscount is a nuisance; **the flattened conditional is the failure
signature this file already names for `surveyor` — fluency.** A claim derived
from a branch must carry the branch. Do not add a rule yet — **two data points,
different shapes; watch run 4.**

---

## Run 3 diagnosis, 2026-08-01 (fresh session) — *named is not surveyed*

Brief: `../DIAGNOSE-run3-misses.md`. Cause, with evidence, and two spec changes
(§4 coverage denominator, §7 gate reads three sources).

### 🛑 Correction: nothing was filtered. D2 and D6 were **recorded and buried.**

The earlier note above says `abaire-16`/`-17` *sampled* the listing and dropped
two keyed repos. **False, and worth correcting** — both are present:
`record-abaire-ecosystem.yml:319` puts `xemu-nxdk_pgraph_tests_results` and
`nxdk_ntrc_dyndxt` inside one record's `subject.name` brace-list of 13 repos.
The record is `R1` and says so honestly (*"not individually deep-dived;
descriptions taken at face value"*). `areas.md` thin-area **#6** says it again:
*"~21 repos — named and one-line-described only, never individually opened."*

⇒ **The proposed "state the filter and its drop count" rule would not have
fired**, because there was no filter. The map declared its own gap twice, in
prose, and `grep ntrc index.yml` returns **nothing** — the index gained no topic
for either repo. **A group record collapses N subjects into one and the reader-
facing surface never learns they exist.**

### The same shape, inward, is what regressed A1 / A3 / B3

| | Run 2 | Run 3 |
|---|---|---|
| Records over `hw/xbox` | **4** (`nv2a-gpu`, `mcpx-apu`, `mcpx-nvnet`, `xbox-platform`) | **1** (`hw-xbox`) |
| Claims over that territory | **27** | **9** |
| Their provenance | `nv2a_int.h:70-110`, `:96-106`, `nv2a.c:334,573-586`, `trace-events:1-30` | `ls hw/xbox/nv2a/`, `ls hw/xbox/mcpx/…`, `ls pgraph/` — **6 of 9 are directory listings** |
| Self-assessed | `R2` | `R2` |

Run 2 found PFIFO's `QemuThread` and its mutex **because nv2a had its own
agent**. Run 3's single record had to cover nv2a + mcpx + platform + SMBus in
nine claims and spent them on the file inventory. **A1/A3/B3 are one file below
that inventory in both runs.**

✅ **Kills hypothesis 1 as stated.** The budget did not shrink: run 3 wrote
**136** claims to run 2's **76**, both across 12 records. It *moved* — `hw/xbox`
27 → 9 while org+abaire went 4 → 36. `reach: both` is where it went, but the
mechanism is the partition, not the reach.

✅ **Kills hypothesis 5's disconfirmer.** `R2` does **not** mean the same thing
in two run-3 records: `ui-xui` at R2 read `debug.cc` and `debug.h` line by line
(and won C1); `hw-xbox` at R2 verified a directory listing. **Same label, 4× the
territory, no way to tell.**

### ⇒ One cause

**A record's subject is a name, not a bounded territory.** When a subject has
many children the record enumerates the children *in prose*, self-awards an
R-level for the enumeration, and **nothing downstream can distinguish "23 things
named" from "23 things surveyed."** Grouping is what hides it; honesty in the
prose does not help, because §7's gate reads `not_determined:` and nothing else.

**Fix 1 (§4)** puts a denominator on it — `R1 · 13 repos listed, 0 opened`.
**Fix 2 (§7)** spends it, alongside the `unexpected:` leads the gate was already
missing. The two compose: one produces the number, the other acts on it.

### Watch, do not spec (one data point each)

- **Fan-out grain.** 4 records → 1 over the same territory is the proximate
  cause of the inward regression, but the brief mandated `both` and the
  orchestrator partitioned by topic noun. **If run 4 with the coverage
  denominator still collapses a large area into one agent, that is the rule to
  write** — one record per territory a single agent can open, not per noun.
- **The flattened conditional** (`hwxbox-3`, 5 SMBus peripherals vs max 3) is
  the fluency signature, unchanged, and note *which record it came from*: the
  one that never opened the file. **Enumeration-depth and confident-prose may be
  the same defect**, not two.

### The coverage denominator closes a loop, so §2 got the third list

Added the same day, and it is the same rule rather than a new one: §2 already
turned `resolution` into a **skip list** and `volatility` into a **re-check
list**. Coverage makes the third — **the open list.** `R1 · 13 listed, 0
opened` is the cheapest work a second pass can find: located already, never
read. Before this, the only machine-readable frontier was the index, and the
index has no entry for a subject that never became a topic — which is precisely
how `nxdk_ntrc_dyndxt` was invisible to a hypothetical run 4 as well as to run
3's reader.

⇒ **Three edits total from the run 3 diagnosis** (§4 coverage, §7 gate,
§2 open list), and the third is a two-line consequence of the first. **Do not
count it as licence for a fourth.**

### ⏸ Held for run 4: a checkpoint at **partition time**, not at depth

Raised by the human: should the survey stop mid-traversal and ask whether to go
deeper? **Probably not there** — the fan-out is parallel, so a mid-run pause
serialises the expensive part, and §7 already argues the orchestrator knows
least mid-run. The gate is at hand-over deliberately.

🎯 **But the partition is chosen at a moment where a checkpoint is nearly
free**, before any sub-agent is dispatched, and that is where run 3's damage was
done: `hw/xbox` got one agent and `abaire's ecosystem` got two, because each is
one noun. One message — *"12 subjects; `hw/xbox` covers nv2a + mcpx + machine
def in a single record; split it?"* — is answerable from the direction alone.

⚠ **Same confound as the splitting rule** (above): the brief mandated a
combined `both` pass, so under-splitting and rationing are not yet separable.
**Run 4 with the denominator in place decides both at once** — if a large area
collapses into one record *again*, the partition needs a human at it and the
checkpoint and the splitting rule fall out of one piece of evidence.
