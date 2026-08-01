# Brief — diagnose why run 3 missed what it missed

**Written 2026-08-01 (s015) for a fresh session.** The scoring is done; this is
the *why*. Start a new thread — do not carry s015's context.

## The question, exactly

**Run 3 scored 6 HIT / 15 MISS / 0 WRONG against a 21-row key. Why the 15?**

🛑 **Not "was the map bad".** It asserted almost nothing false. The question is
**what the process did instead of finding those rows** — which is a different
question and has to be answered from the run's own artefacts, not from vibes.

## Read these, in this order

1. `SCORE-xemu-architecture-2026-08-01-run3.md` — the row-by-row, the validity
   deviations, and the fix-by-fix table at the end. **This brief assumes it.**
2. `ORACLE-xemu-architecture.md` — the 21 rows and their tiers.
3. `~/git_repos/xemu-map-temp-rerun/` — the map itself. `BRIEF.md` first
   (it is *not* a neutral brief — see below), then `areas.md`, then
   `open-questions.md`, then `claims.md`.
4. `surveyor/SKILL.md` §0, §2, §3, §7 and `cartographer/SKILL.md` §5 — the rules
   that were supposed to fire.
5. `SCORE-xemu-architecture-2026-08-01.md` — run 2, for the diff.

## The four misses worth explaining (the other 11 are cheap to explain)

| Miss | Why it is interesting |
|---|---|
| **A1 / A3 / B3 regressed** — which thread renders, the pgraph locks, the surface→texture gate | All three were *near misses* in run 2 (right neighbourhood, wrong file). In run 3 they are **absent entirely**. Same subject, same commit, better spec. **Something bought that depth and spent it elsewhere.** |
| **D2 / D6 dropped from a fetched listing** | `gh api users/abaire/repos --paginate` was run and cited **four times**. The output contains both missed repos. `abaire-16`/`-17` describe themselves as a *selection* by description. **The enumeration succeeded and the selection lost the rows.** |
| **C3 — the 250 ms FPS window** | Missed in both runs, but *differently*: run 2 cited a line range **containing** the answer; run 3 cited a **narrower** range that excludes it. Two different reading behaviours, same outcome. |
| **The six `unexpected:` leads, undispatched** | The cheap-band gate fired correctly on `not_determined:` and never looked at `unexpected:`. One of those leads was `F021`. |

## Hypotheses to test — and what would kill each

Do **not** start from a favourite. Each has a cheap disconfirmer.

1. **`reach: both` in one pass traded inward depth for outward breadth.**
   *Kills it:* if `areas.md`'s per-area resolution shows `hw-xbox` at the same
   R-level as run 2 with comparable claim counts, the budget did not move and
   this is wrong. **Check the counts before believing the story.**
2. **The pre-seeded `BRIEF.md` redirected effort.** Its *"Ground truth already
   established (do not re-derive)"* section is long, names the org repos, the
   test dirs, and the CI layout. *Kills it:* if the inward records still opened
   the files that ground truth pointed at, the brief added rather than replaced.
3. **Sub-agent fan-out fragmented the inward areas.** 12 records, and threads/
   locks/surfaces span `hw-xbox` × `ui-frontend`. *Kills it:* if one record's
   scope plainly covered `pfifo.c` and simply did not open it, the seam is not
   the cause — attention is.
4. **The schema has no slot for "what I chose not to record."** A filtered
   enumeration and a complete one look identical in a record. *Kills it:* find
   any record that does declare a drop-count. If the form exists and went
   unused, it is discipline; if the form does not exist, it is the schema.
5. **The `resolution: R0-R3` field is self-assessed and uncalibrated.** Records
   claim R2 for areas that are one file deep. *Kills it:* compare R2 areas
   against each other — if R2 means the same thing in `build-system` as in
   `hw-xbox`, the field is fine.

## Ground rules

- 🛑 **The subject is read-only.** `~/git_repos/xemu` is the human's build clone
  on `master` @ `cbffb57d08`. Reads only — no checkout, no writes. Same for
  `xemu-map-temp-rerun/`, which is evidence.
- ⚠ **`~/git_repos/the-xemu-cartographer` is NOT off-limits for this task** (the
  blindness condition applied to the *run*, not to diagnosing it) — but you only
  need its `findings/F019`, `F020`, `F021` and `sessions/015`, and reading more
  costs context for nothing.
- **Findings about the skills go here** (`surveyor/NOTES.md`,
  `mapping/NOTES.md`, `EVALUATION.md`). **Findings about xemu go to
  the-xemu-cartographer.** Never both.
- **Deliverable: a cause with evidence, and at most two spec changes.**
  `surveyor/SKILL.md` went 291 → ~450 lines in a day; `mapping/NOTES.md` already
  carries a standing note to consolidate before adding. **A diagnosis that ends
  in three new rules has not diagnosed anything.**

## Already known, do not re-derive

- Two off-key false claims: `subprojects/*.wrap` **35 vs 36**; `hwxbox-3`
  *"registers 5 SMBus peripherals"* vs **max 3** (an `if/else` on
  `video-encoder`, `hw/xbox/xbox.c:299-313`).
- Conservation passed: 30 `not_determined` in, 30 out, hand-counted.
- The cheap-band gate fired, the human approved, 11 resolvers ran, **4 index
  entries superseded**. That part works.
- `D8` (the CI negative) is the map's best work, reproduced from run 2 with four
  independent greps and the `[docs]`/`[source]` tension preserved.
