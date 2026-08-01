---
name: cartographer
description: Compile survey records into a map — an append-only body of claims plus a mutable, authoritative index that resolves a topic to the current claim, tracks resolution per area, and marks contested findings. Use when survey records need turning into a map, when a map's index needs bringing current, when findings have accumulated and nobody can tell which are still true, or when a long document has begun superseding itself.
maturity: draft
---

# Cartographer

**Speaks map format v1.** Consumes survey records from `surveyor`. Produces the
map and its index. Renders nothing — that is `visualiser`.

You are the drawing office. Field parties took the measurements; you decide what
the map says, which claims are current, and where the map is thin.

## The map is three things with different rules

| Part | Mutability | Purpose |
|---|---|---|
| **Claims** | **Append-only, immutable** | What was found, with provenance. Never edited, never deleted |
| **Index** | **Mutable, authoritative** | Topic → the *current* claim. The mandatory entry point |
| **Areas** | Mutable | The territory, each carrying a **resolution** |

🛑 **Supersession is an index operation.** A later claim that overturns an
earlier one does not edit or delete it. The claim stays; the *index entry* is
repointed. The reasoning trail survives and only the answer moves.

This exists because a long body of claims supersedes itself, and **a stale claim
with correct provenance passes every other check anyone could apply to it.**
Searching returns whatever matched first; the index returns what is true now.

## 1. Read before writing

- Ask where the map lives, **and record the answer in that location's own
  `README.md`** so the next run reads it instead of re-asking.
- **Read that location's own `CLAUDE.md` / `README.md` every run** — domain
  conventions belong to the domain, not to this skill.

### 🛑 There is probably already an entry point. Ask before you become one

The index below calls itself the mandatory entry point. **Most established
domains already have one** — a `STATUS.md`, a README, a wiki homepage, an ADR
index, a runbook — and **two authoritative entry points is the exact failure this
skill exists to prevent.**

So do not resolve that silently. Ask which of three this is, and write the answer
into the map location's `README.md`:

| | The index… |
|---|---|
| **Replaces** | The incumbent is retired or becomes a pointer to the index |
| **Feeds** | The incumbent stays authoritative for humans; the index is compiled *into* it, by the human or on request |
| **Beside** | Both stand, for different audiences. **Then say plainly which one wins on a disagreement** |

**All three are legitimate. Picking one without asking is not** — and note that
the incumbent is usually the artefact the human actually relies on day to day.
- **Read the existing index in full** before ingesting anything. You cannot
  detect supersession against a map you have not read.
- **If the map declares a format version other than v1, stop and say so.**
- Note the **direction** the records were surveyed against. If it differs from
  the map's recorded direction, say so before compiling — the survey may need
  re-firing rather than merging.

## 2. Ingest each record

For every claim in every record, decide which of these it is:

| Case | Action |
|---|---|
| **New** — no index entry for this topic | Append the claim. Create an index entry |
| **Corroborates** — agrees with the current claim | Append. Add it as supporting evidence; leave the entry pointed where it is |
| **Supersedes** — later, better-grounded, and incompatible | Append. **Repoint the index.** Record what it superseded and why |
| **Contests** — incompatible, and you cannot honestly say which wins | Append. **Mark the entry `contested` and point at both.** See below |
| **Raises resolution** — same area, more detail | Append. Update the area's resolution level |

### 🛑 Only `claims:` may become an index entry. Free text may not

A record's `unexpected:` field — and any other free text in it — carries **no
grounding, no tier, no provenance, no `n`**. The claim schema is where a
surveyor's discipline is enforced, and content that never passed through it has
never been checked.

**So it does not enter the index as a claim.** Not as `current`, not as a
correction, not as a numbered entry:

| Where it came from | The most it may become |
|---|---|
| `claims:` | `current` / `contested`, per the table above |
| `not_determined:` | A `not_determined` entry with its `resolved_by` |
| **`unexpected:` or any free text** | 🛑 **A lead** — `not_determined`, with the record's own hedge **preserved verbatim** and `resolved_by` naming what would check it |

⚠ **Preserve the hedge word for word.** A record that says *"run 1 = 15,163…
recording, not concluding"* is being honest; the failure is a compile that keeps
the number and drops both qualifiers. **If you find yourself paraphrasing a
hedge, you are laundering it.**

🛑 **This binds hardest on the corrections you report to the human**, because
that section carries the most authority in the whole map. A correction resting
on free text is **`unverified`**, and says so.

### ⚠ A derived statistic with no `n` is not a claim, whatever it looks like

If a record asserts that two values **differ** and does not say what each was
computed over, **you cannot promote it**. Correct provenance says where a number
came from; it never says how many of it there were.

Enter it as `not_determined`, `resolved_by` = recompute over the full set. **The
compile cannot check this itself** — you do not hold the underlying data — and
**that is the reason the entry must not claim to have been checked.**

### 🛑 `contested` is a first-class state, not a failure

If two claims disagree and the evidence does not settle it, **the index must say
so and name both**. Forcing a single current answer manufactures confidence in
exactly the case where the map knows it is unresolved — and that is the most
dangerous entry a map can contain.

An entry may also be `contested` when one claim is `[source]` and another is
`[inferred]` but from a later, better-informed position. Record the tension;
do not launder it.

## 3. Prefer the better-grounded claim, not the later one

Supersession is **not** chronology. When deciding which claim is current:

1. **Grounding wins**: `[source]` over `[docs]` over `[inferred]`.
2. **Then tier**: a T4/T3 claim beats a T1 claim about the same thing.
3. **Then recency**, and only then.

A recent `[inferred]` claim does not supersede an older `[source]` one. If it
appears to, that is a `contested` entry, not a repoint.

## 4. The index entry — map format v1

```yaml
topic: <the question someone would actually ask>
status: current | contested
current: <claim-id>            # or, when contested:
contested_between: [<claim-id>, <claim-id>]
area: <which area of the map this sits in>
resolution: R0 | R1 | R2 | R3
volatility: static | slow | live
last_checked: <YYYY-MM-DD>
supersedes:
  - claim: <claim-id>
    reason: <why it stopped being current>
```

**Key entries by the question, not by the document.** *"What is the build
route?"* is a topic; *"`FINDINGS.md`"* is not. Staleness happens at claim level, so an
index keyed by document cannot repoint at the granularity where the problem
lives.

## 5. Keep the areas honest

For every area the map covers, record its resolution and **draw blank as blank**.
An unmapped area gets a boundary and no content — omitting it entirely reads as
"there is nothing there", which is a claim nobody made.

Maintain a short list of **thin areas**: where the direction needs more than the
current resolution supplies. That list is what tells the user whether to re-run
the surveyor, and it is one of the most useful things the map produces.

### 🛑 Record which **reach** each pass covered — surveys arrive in passes

Records carry a `reach` (`inward` / `outward` / `both`). **Carry it into the
areas file and state it in the index header:** *"inward pass compiled
2026-01-01; outward never run."*

**Thin and unsurveyed are different failures and take different actions.**

| | What it means | What it needs |
|---|---|---|
| **Thin** | Looked at, not deeply enough. Carries a resolution | Re-survey the area |
| **Unsurveyed by reach** | **Nobody ever looked.** Carries no resolution, no record, no `not_determined` | A **new pass** at the missing reach |

🛑 **The second row is invisible unless you write it down.** It leaves no trace
in the records — an `inward`-only compile is *complete and internally
consistent* with the entire ecosystem absent, and reads as though the subject
stands alone. **Only the pass log says otherwise.**

✅ **A later pass is additive, and needs no special handling:** claims are
append-only and the index is keyed by topic, so an outward pass adds topics
beside the inward ones. **Update the pass log, and never re-open settled inward
entries just because a new pass arrived.**

## 5b. 🛑 Every `not_determined` bubbles up. None may be dropped

The index needs a **register of open questions**, maintained exactly like the
contested register, carrying every `not_determined` from every record:

| Question | Which record raised it | What would resolve it | Cost |
|---|---|---|---|

**This is not optional, and it is not a summary.** Every `not_determined` in
every ingested record appears, or you say in the hand-over which you dropped and
why. **Silently dropping them is the default failure** — they are the one part of
a record that names something *absent*, and absence does not attract a topic
entry to attach itself to, so it falls out of a topic-keyed index unless a
register catches it.

🛑 **Measured on the first real compile: 49 open questions went in, ~4 came out.**
The dropped ones included a record's doubt about *how a dispersion figure was
defined* — raised in the same compile that promoted a comparison with no `n`.
**The pipeline had already noticed, and the compile threw the notice away.**

### Why this register is the family's real output

A map's value is not only what it settled. **The open questions are the survey's
verdict on itself** — they are where the next work is, and they are the part no
amount of re-reading the corpus regenerates, because the reader who could have
asked the question has already moved on.

- **Rank them by cost**, cheapest first. "Recompute this from data already on
  disk" and "run a three-week experiment" must not be adjacent and undifferentiated.
- 🛑 **Price by the operation, never by its category.** The band is *how many
  calls and how long* — **not** "local vs external", "code vs web", or "in scope
  vs out". **One non-mutating call is cheap wherever it lives.** Do not inherit
  a record's own banding uncritically: **re-price each resolver as you ingest
  it**, because you are the last reader before the human, and the band is what
  they will believe.
  ⚠ **Measured:** a compile carried *"direct fetch of the org's repository
  listing"* into its **most expensive band, 34th of 35** — one API call, ~2
  seconds, and the highest-value unresolved item in the whole register.
- 🎯 **Surface the cheap band at the top of the hand-over, not just in the
  file.** A register whose first tier is *"three things, one call each"* gets
  acted on; the same three at position 34 do not.
- **Carry `resolved_by` verbatim.** A question without its resolver is a
  complaint. **The resolver is the deliverable.**
- ⚠ **A question the compile *created* counts too** — anything you marked
  `unverified` or demoted to a lead under §2 belongs here, or your own caution
  disappears the moment you stop typing.

## 6. Volatility is per claim, not per source

Two claims from the same source on the same day can decay at completely
different rates — a live count and a closed historical record. Carry
`volatility` on the claim and surface `live` entries whose `last_checked` is old
**before** anyone quotes them.

## 7. The read-back gate — do not skip this

🛑 **Adherence fails under context pressure; verification catches it.** A
followed protocol still produces a bad map at the end of a long session.

Before finishing, **re-read the index you have just written, cold, as if you had
not been present**, and check:

- Does it answer the **direction**? If not, say so explicitly.
- Does every entry resolve to a claim that exists?
- Is any entry `current` where the evidence is genuinely contested?
- Would someone entering here be sent to the newest claim on each topic?
- 🛑 **Count them.** How many `not_determined` items went into this compile, and
  how many are in the open-questions register? **If the second number is smaller,
  name every missing one or put it back.** Do this by counting, not by
  impression — the first real compile lost ~45 of 49 and nobody noticed.
- 🛑 **Trace every entry back to its field.** Does any `current` entry — or any
  correction you are reporting — rest on `unexpected:` or other free text? Does
  any comparative number lack an `n`? **Those are the entries you will be least
  able to defend and most likely to be quoted on.**
- ⚠ **Which entry would you be most embarrassed by if the human recomputed it?**
  Go and mark that one, now. Compiling cannot verify data, so the map's job is to
  say plainly which entries were never verified.
- Is anything asserted in your summary that is **not written in a file**? If it
  matters after this turn, it goes in the map — not in the reply.

## 8. Rules

1. **Claims are append-only.** Supersede by repointing the index; never edit or
   delete a claim.
2. **The index is the entry point.** When answering from the map, look up the
   topic — never conclude from a search across claims.
3. **Never smooth over a gap.** "Not determined", plus what would resolve it.
   🛑 **And that gap must reach the index's open-questions register** (§5b). A
   `not_determined` recorded in a survey and dropped at compile is smoothed over
   just as thoroughly as one never written — more so, because someone did the
   work of noticing and the map now implies nobody did.
4. **Every claim keeps its grounding, tier, provenance and accessed date**
   through compilation. Compiling never strips provenance.
5. **Two records disagreeing about the same subject is a finding**, not a merge
   conflict. Surface it; it is one of the main things compilation is for.
5b. 🛑 **Compiling never upgrades a claim's standing.** An entry is at most as
   checked as the field it came from. Free text stays a lead; a statistic with
   no `n` stays unverified; a hedge survives verbatim. **The index inherits
   authority from its claims — it cannot confer any.**
6. **Compile what exists before asking for more.** Re-reading material already
   collected outperforms new collection often enough to be a gate.
7. 🛑 **Write only to the map location. Never edit the domain's own documents or
   source** — not the README you read for conventions, not the status page, not
   the findings the records were compiled from, however stale they look. **Report
   what is stale; let the human change it.** Compiling a map must never mutate
   the territory it describes, and the artefacts you are most tempted to correct
   are the ones the human relies on most.
8. 🛑 **The territory is a READ-ONLY ARTEFACT. Compile from it; never change
   it.** Rule 7 stops you rewriting its *files*; this covers everything else —
   **its history, its working tree, and anything sent outward on its behalf.**
   **A property of the subject, not a list of banned commands:** if an action
   alters bytes in the territory, it is out of scope whether or not it is below.
   - **Read-only git is a legitimate compile instrument** — `log`, `show`,
     `blame`, `diff`, `rev-parse` for pinning a SHA, dating a claim, checking
     whether a file moved. Use it freely.
   - **Never git-write:** `commit`, `add`, `checkout`, `switch`, `branch`,
     `stash`, `merge`, `rebase`, `reset`, `restore`, `clean`, `tag`, `push`.
   - **Never write by any other route:** no files created in the territory, no
     builds, installs or formatters that write into it, no deletions.
   - 🛑 **Nothing leaves the machine** — no PRs, issues, tracker comments or
     publishing, however clearly the territory is broken. **§ Corrections owed
     is where that goes; the human applies it.**
   - 🛑 **Committing the map into the surveyed repo is the specific accident to
     avoid** — your output in someone else's history, one `push` from being
     public and attributed to them. **If the map location would sit inside the
     territory, say so and ask for somewhere else.**

## 9. Hand over

Report in a few lines: what was compiled, what changed status, what is now
`contested`, which areas are thin, and what the direction still cannot answer.

If the user needs to *see* the map rather than read it, that is `visualiser`.
