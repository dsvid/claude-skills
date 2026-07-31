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
