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

✅ **IMPLEMENTED same day, in all three skills** — the human overruled my
"wait and observe", correctly.

- `cartographer` §1 — a new subsection: *there is probably already an entry
  point, ask before you become one*, with the replaces / feeds / beside table,
  and the answer written into the map location's `README.md`.
- `cartographer` §8 rule 7, `surveyor` §6 rule 7, `visualiser` §5 — **write only
  to the map location; never edit the domain's own documents or source.**
- `cartographer` §1 also now **persists the map location** rather than re-asking
  every run.

⚠ **My reason for deferring was bad and is worth recording as a method note.** I
argued the first run was "the only clean observation of the drafted version". But
the observation at stake was merely *does it ask unprompted about `STATUS.md`* —
low value — while the downside was **it edits the human's day-7 skim target**.
I had also already changed `surveyor`'s `SKILL.md` before its own first run
without the same hesitation. **"Preserve the experiment" is a good instinct that
was pointed at a worthless experiment**; the cost of being wrong was asymmetric
and I did not weigh it.

**Not implemented:** `capture-learning`'s persistence *mechanism*
(`bd remember` / `bd recall`). That is a tool dependency the mapping skills
should not assume; writing the answer into the map location's own `README.md` is
the portable equivalent and is what §1 now says.

### 2026-07-31 — pre-run: three holes in the v1 contract (raised by the human)

**The human asked whether we "set the cartographer up to expect an index as
input but didn't set the surveyor up to output one."** The flow is *not* broken
that way — **the cartographer WRITES the index (§4); `surveyor` §2 and
`visualiser` §1 READ it.** It is the cartographer's own prior output, read back
on later runs. That chain is coherent.

**But the question is pointed at something real. Three holes, watch all three on
run 1:**

1. 🛑 **The index points at `claim-id`s the surveyor never emits.** §4 entries
   are `current: <claim-id>` / `contested_between: [<id>, <id>]`. **The
   surveyor's record schema has no id field** — claims arrive as anonymous
   bullets under `claims:`. So ids must be minted at ingest, and **neither skill
   says so.** This is the closest thing to the human's hypothesis and it is a
   genuine v1 contract break: *the surveyor emits nothing the index can point
   at.* Cartographer owning id-minting is a defensible design (it owns the
   append-only claim store) — but implied, it produces two incompatible id
   schemes the moment two runs start cold.
2. **Cold start is undefined.** §1: *"read the existing index in full before
   ingesting anything"* — **no branch for there isn't one**, which is run 1.
   `surveyor` §1 handles the equivalent ("if there is none, ask where it should
   go"); this skill does not.
3. **Nowhere says where the index physically lives.** §4 defines the entry
   *schema*, not the file. One file? One per area? `_index.yaml`? All three
   skills say "read the index"; none says what to read.

⚠ **Deliberately NOT fixed before run 1 — and this time the reasoning is
different from the write-boundary case above.** There, deferring risked damaging
the human's own artefact for a worthless observation. Here **nothing is at risk**,
and how the skill invents ids and copes with an empty index is **the informative
part**: it tells us whether `map format v1` is a real contract or merely a shared
vocabulary. **If it improvises all three coherently, the gaps are documentation.
If it stalls or invents something the surveyor could never produce, they are
design.**

### 2026-07-31 — "why doesn't the surveyor write the index?" (the human, pre-run)

**The human asked this cold, without having seen s009's §K kill criterion. They
arrived at it independently.** That is worth more than the argument below: the
split is not self-evident to a reader who has the skills in front of them.

**The real case for the cartographer owning the index:**

1. **The surveyor is *n* agents each seeing a slice; the index needs one writer
   that sees everything.** §3 fans out. Concurrent writers to the one
   authoritative file, each blind to the others, cannot resolve supersession —
   which is inherently cross-record.
2. **Supersession is cross-*run*, not just cross-record.** A survey holds only
   its own records. "This overturns one from three months ago" needs the
   accumulated map. The surveyor reads the index but sees only its conclusions,
   never the other run's reasoning.
3. **The operations decouple both ways.** Compile without surveying (records
   arrive from elsewhere; or the same records need re-adjudicating under a
   changed rule). Survey without compiling (emit now, index later, possibly by
   someone else).

🛑 **The honest counter, which is the human's point: every one of those is
satisfied by "phase 2 of the surveyor."** Argument 1 says only that *the parent
must do it after fan-out* — and the parent **is** the surveyor. This proves a
separate **step**, not a separate **skill**.

### The discriminator to settle it

> **Does the compile ever run on records the surveyor did not gather?**

**Yes** — someone else's records, records from months ago, records re-adjudicated
under a new rule ⇒ **skill**, it has a life independent of any survey.
**No** — always survey-then-immediately-compile, same agent, same session ⇒
**phase**, and the split costs a handoff and a file format for nothing.

⚠ **Today looks like "yes" but does not count.** The records cross a session
boundary only because I chose to clear the context to avoid contaminating the
scoring. **A manufactured instance is not evidence.**

⚠ **And one signal already points at "phase":** the surveyor alone surfaced all
11 traps and 6 contested claims before any compile ran (`surveyor/NOTES.md`,
first dogfood entry). **If the compile adds nothing on top of
`/map/survey/README.md`, that is the answer.**

---

## First compile result — 2026-07-31 (s011, cold session)

Run against `/map/survey/*.yaml` (24 records, 296 claims) in a session that had
read only the s010 log, `questions/README.md`, `_direction.md` and the survey
README before starting. The five checks were pre-registered in
`sessions/010-…:116-131`. Answered here with what happened, not what should have.

### Check 1 — did it beat the survey report?

**Yes. Three items, of which one is a class the survey could not have found.**

- **J1 — a LIVE claim recorded as DEAD.** `questions/README.md`:483 states
  "F009 Claim 1's 'class, not single title' is REFUTED". F009 Claim 1 is
  "#480 conflates a fixed regression with an unfixed baseline defect", is on
  F009's own survivor list, and `STATUS.md`:842 still relies on it as live. The
  refuted text is F009's Q8.1 *bearing statement* plus Claim 3. **The two repo
  files now contradict each other about one claim's status.**
  The survey report has all the parts (`f009#4` says the parent brief's framing
  is wrong) but never joins them, and never names `questions/README.md`:483 —
  its trap table T1 flags a *different* defect on a *different* line of that file.
- 🔥 **This is the inverse of the survey's whole trap class.** T1–T11 are all
  *dead claims reading as live*. This is a *live claim reading as dead* — and
  **"read forward" is what produces it.** A discipline that resolves every
  conflict in favour of the later document manufactures this error whenever the
  later document misnumbers. The surveyor's direction cannot catch it; the
  index can, because the index is keyed by claim and must hold one status per
  claim.
- **J2 — the corpus's own ledger has a stale row.** `F017`:104-116 lists
  "draw-command COUNT as the cap — dead (F012 Claim 8)". `f012#13` was refuted
  by `f013#4`. Not in the survey report.
- **J5 amplified, J3, J6** — see `/map/index.md` § J.

**The kill criterion did not fire.**

⚠ **This is the sharpest argument yet for a MUTABLE index, and it is a different
argument from the one that motivated the skill.** The original case was
*append-vs-revise*: documents grow by appending, so tables go stale. This case is
**adjudication**: two documents disagree about one claim's status, and only a
claim-keyed structure forces them to meet. An index that must hold exactly one
status per claim surfaces the collision automatically; a document-keyed index
never can.

⚠ **But note how J1 was found: by cross-reading two records from two different
sub-agents (`f009` and `status`) against a third file.** That is compile work
by nature. It is *not* evidence that the compiler must be a separate skill —
a surveyor parent doing phase 2 would have the same records.

### Check 2 — did it touch `STATUS.md` or any domain document?

**No.** Verified by `git status --short` and `git diff --stat`, not recollection:
the working tree shows exactly three untracked files, `map/README.md`,
`map/areas.md`, `map/index.md`, and an empty diff. Rule 7 held.

**The rule cost something and that is worth recording:** the compile found six
corrections it was forbidden to apply, and had to invent a place to put them
(`index.md` § J, "Corrections owed to the territory"). **§ J should be in the
skill.** A compile that finds staleness and has nowhere structured to report it
will either leak into the territory or lose the finding in prose.

### Check 3 — cold start, with no existing index

**It asked; it did not stall or improvise.** §1's "read the existing index in
full" was unsatisfiable — there is none — so the run went to the
replaces/feeds/beside question directly and put both decisions to the human in
one call before ingesting anything. Answers: **map at `/map/` root**, and
**FEEDS** (`STATUS.md` stays authoritative for humans, index compiles into it).
Both are written into `/map/README.md`, so the next run reads them.

⚠ **Gap in the skill:** §1 says "read the existing index in full" and says
nothing about the first run, where there is none. It happens to fall through to
the right behaviour because the replaces/feeds/beside question is right below
it. **Make the bootstrap case explicit** — a first run has no incumbent index
but always has an incumbent *entry point*, and that is the thing to ask about.

### Check 4 — claim ids

**Minted, coherently, and reproducibly — by refusing to mint a counter.**

Scheme: `<record>#<n>` — the record's own `record:` slug and the claim's 1-based
position in its `claims:` list. A global `c001…` sequence was considered and
rejected: it renumbers on insertion, so a second run over a changed record set
would not reproduce it. `<record>#<n>` is derivable from the YAML alone, by any
run, in any order, with no registry. **A second run reproduces it exactly.**

Consequence worth keeping: **no transcribed claim register was written.** The
records already carry statement/status/grounding/tier/provenance/accessed/
volatility; copying 296 of them into a second file creates a lossy duplicate
that can drift from its source. **The records ARE the append-only claim body.**
The map is then three files — README (conventions + the two decisions), index
(mutable, authoritative), areas (territory + thin + blank).

🛑 **This is a real change to the v1 contract and the skill should state it.**
§4's index entry uses `current: <claim-id>` as if a claim register exists
somewhere. It does not need to. **Address claims in place.**

⚠ **Cost of the scheme:** it is only stable if records are append-only *within*
themselves. If a surveyor inserts a claim mid-list on a re-survey, every id
below it shifts. **Records must be append-only at claim level, not just at file
level** — the skill does not currently say so, and this is now load-bearing.

### Check 5 — the discriminator (does the compile run on records the surveyor did not gather?)

**Still unanswered, and today does not answer it.** The handoff was manufactured
by clearing context, exactly as the pre-registration warned. Same repo, same
day, records gathered by sub-agents of the previous session.

**What this run does contribute to the question:** the compile produced findings
the survey did not (check 1), which kills the strongest "phase" argument — that
compilation is bookkeeping. **It does not establish independence.** J1 was found
by cross-reading records from two different sub-agents, and a surveyor parent
doing phase 2 has exactly the same material.

⇒ **The honest position after run 1: the compile is a distinct OPERATION with
its own output, provably not a re-statement. Whether it is a distinct SKILL is
untouched.** The discriminator still needs a real instance — records this agent
did not commission, or a re-adjudication of old records under a changed rule.

---

## Run 2 — 2026-08-01, the scored re-compile (same 24 records, cold session)

Pre-registered by the human at the-xemu-cartographer s012 close, scored by
`grep -c`, not by judgement. **Result: §5b took. 49 `not_determined` in,
49 out** (run 1: 49 in, ~4 out). Register at that repo's `map/index.md` § L.

### What made the difference, concretely

**Not the rule text alone — the rule plus a mechanical extraction step.** The
compile did not read the records and remember the questions; it ran a parser
over the YAML that printed every `- question:` with its record and its
`resolved_by`, numbered 1–49, *before* writing anything. Then it wrote the
register from that dump and verified with a script that every question's first
45 characters appear in the finished index.

⇒ **Candidate skill change: say this in §5b.** "Extract the `not_determined`
items mechanically into a working list before compiling, and diff the finished
register against that list." A rule that asks the compiler to *remember* 49
items across a long compile is asking adherence to survive context pressure —
which is the exact failure §7 exists to catch. **Give it a tool, not a
reminder.** Cost: two short scripts, ~3 minutes.

### 🔥 The rule gap this run found: §2's free-text rule is ingest-only

The secondary criterion was *"nothing from the 24 `unexpected:` blocks appears
as a `current` entry or a correction."* This run passed it on **its own** output
— and then found **two violations already sitting in the index from run 1**:

- a host-CPU correction ("6 cores, and STATUS.md reasons from 8") resting
  entirely on a record's `unexpected:` block, not on any claim;
- a "the only matched-duration fast/slow pair the exercise owns" line, likewise.

**Neither was catchable at ingest, because ingest was over.** §2 and rule 5b
tell you what free text may *become*; nothing tells a later compile to
**re-audit entries a previous compile already wrote.**

⇒ **Candidate skill change: §7's read-back gate should trace *existing* entries
back to their field, not only new ones.** The gate already says "trace every
entry back to its field" — but in practice a re-compile reads the index as
context (§1) and treats it as settled. **Say explicitly: the index you read in
§1 is input to be audited, not authority to be inherited.** A map that is
compiled more than once needs the free-text rule to be a *standing* invariant.

### Second-order: the register is where the map's own doubt goes to survive

The five compile-created questions (`L-b`) include the compile's nomination of
**the entry it would be least able to defend** (§7 asks for this; nothing said
where to *put* it). Run 1 had nowhere to put it and it evaporated. **§7's
"mark that one, now" should name the open-questions register as the destination
— otherwise the caution is written in the reply and lost**, which is the same
failure as dropping a `not_determined`.

### Check 5 — the discriminator: still unanswered, and this run is weaker on it

Same repo, same records, and the compiler was **told the pre-registration and
the score criteria before it began**. That is the opposite of a blind run.
**Passing here is the floor.** `EVALUATION.md` still needs a fresh domain.
