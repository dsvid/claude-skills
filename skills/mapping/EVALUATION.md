# Evaluating the mapping skills

**How we decide whether `surveyor`, `cartographer` and `visualiser` work.**
Not how to use them — that is each `SKILL.md`. Not what we learned building them
— that is `NOTES.md`.

🛑 **This is not the read-back gate.** Each skill's `SKILL.md` carries a
self-check the agent runs on its own output, during the run, asking *"did I do
the job?"* **This file is scored by someone else, after the run, and asks
*"does the skill work at all?"*** Different runner, different time, different
subject. **If a check ever appears in both, delete it from one** — content lives
in one place or neither is authoritative.

**One file, family-level, deliberately.** The load-bearing checks are
*conservation across the handoff* — `not_determined` in versus register out spans
`surveyor` → `cartographer`, so no single skill can own them. Per-skill files
would each hold a fragment and the pipeline checks would have no home.

---

## 🛑 "It ran and produced a nice map" is an unscoreable outcome

It is also the default one. **Decide the criteria before the run, and write the
decision the map is supposed to inform.**

**"Is the skill good?" is not one question.** Evidence from the first real run
(2026-08-01, the-xemu-cartographer): ~200 pointer claims held up, **one derived
statistic was false**, and **~45 of 49 open questions vanished at compile**.
Three failure modes, three tests. **A single global judgement scored that run a
success.**

⚠ **The generality problem, stated plainly:** all three skills were derived from
one repo with its corpus in front of the author. **Working there is not evidence
of generality.** But a fresh domain has **no oracle** — which is exactly what made
the 2026-08-01 catch possible and what a new domain takes away.

## The five tests

| # | What it tests | How | Cost | Needs judgement? |
|---|---|---|---|---|
| **A** | **Conservation** — did anything silently vanish? | Count `claims:` in vs index entries out; `not_determined` in vs register out; every entry resolves to a real claim id; no entry traces to free text; every comparative statistic carries `n` | Minutes, mechanical | **No** |
| **B** | **Claim precision** | Sample 20 claims, verify each against source | An hour | Barely |
| **C** | **Confabulation** | Two cold surveys, same direction, independent. **Adjudicate only the diff** | One extra run | Only on the diff |
| **D** | **Correctness against an oracle** | Choose a domain with a document that can score the map — CHANGELOG, ADR set, postmortem, a maintainer — and **withhold it from the run** | Domain choice | No |
| **E** | **Usefulness** | **Pre-register a decision you actually need to make. Did the map change what you did?** | Free | Yes, and it is the point |

### The three that matter most, and why

- 🛑 **Stratify sample B by claim type, or it will hide exactly what it should
  find.** Today: pointer claims ~100% correct, derived statistics 1-for-2. An
  unstratified sample of 20 draws ~19 easy ones and reports 95%. **Sample 10
  pointer-claims and 10 derived/inferred claims separately and report two
  numbers.** One aggregate number is how this defect stayed invisible.
- ✅ **C is the cheapest real signal on a domain with no oracle.** Where two
  independent surveys agree, weak evidence; **where they disagree, at least one
  is confabulating**, and you only have to hand-check the disputed points. It
  turns "verify a whole map" into "verify a diff".
- 🛑 **E is the one people skip and it outranks the rest.** A map that is
  accurate, conserved, and changes nothing about what you do next **has failed**,
  and no amount of A–D detects that. Write the decision down before the run.

### ⚠ The limit to state up front: a fresh domain cannot test adjudication quality

A–D test **precision and conservation**. They do not test the thing the
cartographer mainly does — **taking a defensible position where the corpus holds
two**. Judging that requires knowing the domain well enough to know who was
right, which a fresh domain by definition denies you.

⇒ **Two runs, not one, and they test different things:**

| Run | Domain | Tests |
|---|---|---|
| 1 | **Fresh, with an oracle** (D) | Precision, conservation, confabulation — the generality claim |
| 2 | **A domain you already know cold** | Adjudication quality — did it side correctly on contested points, and did it surface any you had missed? |

**Neither alone settles it.** Run 2 is not a weaker version of run 1; it is the
only one that can score the skill's central operation.

### What would kill the skills rather than tune them

- **A fails** ⇒ the spec fixes made 2026-08-01 did not take. Structural, not a
  tuning problem.
- **C shows the two surveys diverging on substance** (not just coverage) ⇒ the
  records are being authored, not gathered. **That is fatal to the family**, not
  a fixable rule.
- **E fails on a domain where A–D all pass** ⇒ the artefact is fluent and
  useless, which the source repo names as **worse than no map**. That is a
  finding about the *format*, not the skills.

---

## Per-skill: what "good" means for each

Derived from where each has actually failed, not from what each aspires to.

### `surveyor`
- ✅ Claims carry grounding, tier, provenance, accessed, volatility — **no exceptions**.
- ✅ **Derived statistics carry `n` and spread in the statement.** Known failure: a median over one run described as the arm's.
- ✅ `unexpected:` holds leads only — nothing falsifiable, no numbers.
- ✅ Records what it could **not** establish, with `resolved_by`.
- 🛑 **Failure signature:** fluency. Records that read well, cite correctly, and assert more than was checked.

### `cartographer`
- ✅ **Conservation.** Claims in vs entries out; `not_determined` in vs register out. Counted, not estimated.
- ✅ Nothing in the index traces to free text; nothing comparative lacks `n`.
- ✅ `contested` used where the corpus genuinely holds two — **not collapsed**.
- ✅ Hedges preserved verbatim through promotion.
- 🛑 **Failure signature:** silent loss. It drops what has no topic to attach to, and a compile that loses things looks identical to one that had less to work with.

### `visualiser`
- ✅ Renders from the index alone; inferred drawn differently from sourced; blank areas drawn blank.
- ✅ Never renders an `unverified` or `n`-less figure as a bare number.
- ✅ Reports what it **could not draw**, and why.
- 🛑 **Failure signature:** a diagram strips qualifiers. Anything it draws looks equally true.

---

## Scores so far

| Date | Domain | Run | A cons. | B prec. | C conf. | D oracle | E useful | Notes |
|---|---|---|---|---|---|---|---|---|
| 2026-07-31 | the-xemu-cartographer | surveyor + cartographer + visualiser | 🛑 **FAIL** — 49 `not_determined` in, ~4 out | ⚠ pointer ~100%, derived **1 of 2** | not run | n/a (self) | ✅ found 11 dead-live claims, 1 live-dead, 1 false claim of its own | Spec fixed 2026-08-01; **fixes unvalidated — no run since** |

🛑 **The table is the point.** Criteria nobody scores are a wishlist. **Add a row
per run, including the runs that went fine** — a fix is not validated by the
session that wrote it.
