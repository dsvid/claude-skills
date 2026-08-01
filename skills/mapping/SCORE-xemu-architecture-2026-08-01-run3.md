# Run 3 score — xemu architecture, re-run against the same oracle

**Scored 2026-08-01.** Map at `~/git_repos/xemu-map-temp-rerun/` (12 records,
`claims.md` + `index.yml` + `areas.md` + `open-questions.md`). Key:
`ORACLE-xemu-architecture.md`, 21 rows. Baseline: run 2, **4 HIT / 17 MISS /
0 WRONG**.

## Headline

| | Run 2 | Run 3 |
|---|---|---|
| Overall | 4 / 17 / 0 | **6 HIT / 15 MISS / 0 WRONG-on-key** |
| T-must (4) | 1 / 3 | **2 / 2** |
| T-should (8) | 2 / 6 | **3 / 5** |
| T-deep (9) | 1 / 8 | 1 / 8 |
| Off-key false assertions | 2 (both counts) | **2 (one count, one flattened conditional)** |
| Conservation | 35 in / 35 out ✅ | **30 `not_determined` in / 30 out ✅**, hand-counted per file |

**+2 rows. Still a fail at the must tier.** The gain is real but narrow: **C1
and D1**, both of which were the specific misses the 2026-08-01 fixes targeted.
**Nothing else moved**, and the inward graphics-pipeline half got *worse*.

## 🛑 Validity — this run is NOT a clean replication of run 2

Three deviations from the oracle's own runbook. Record them before reading the
numbers.

1. 🛑 **The brief pre-seeded the answer.** `BRIEF.md` § *Ground truth already
   established (do not re-derive)* hands the run the full `xemu-project` org
   listing **including `xemu-test` and `nxdk_pgraph_tests`** — the exact items
   run 2 missed and was marked down for. **D1's HIT is substantially gifted.**
   The oracle's validity condition is *"answer 'nothing — treat this as a cold
   read'"*; that was not done.
2. 🛑 **It surveyed `~/git_repos/xemu`**, which the runbook forbids by name
   (*"Do not survey `~/git_repos/xemu`"*). ✅ Mitigations held: the clone was on
   `master` @ `cbffb57d08` — the exact commit the key was written against — and
   read-only discipline held (no writes observed, brief carried the rule).
3. ⚠ **Reach was set to `both` and the outward half was made obligatory** by the
   brief (*"Outward enumeration is obliged, not optional"*). That is the new
   `reach` feature working as designed, but it means the outward gain cannot be
   attributed to the skill alone.

⇒ **Read run 3 as "did the fixes fire when pointed at them", not "did the score
move on equal terms".** It answers the first well and the second not at all.

## Row by row

| # | Row | Tier | Run 2 | Run 3 | Evidence |
|---|---|---|---|---|---|
| A1 | renders on `pfifo_thread` | **must** | MISS (near) | **MISS** ⬇ | **Regressed.** Run 2 at least found PFIFO's `QemuThread` in `nv2a_int.h`. Run 3 has *no* claim about nv2a threading at all — the only thread claim in the map is `uifrontend`'s `qemu_main`/`vblank-timer`/BQL (claims.md:269). Would not have prevented s004. |
| A2 | TCG, no accelerator shipped | should | MISS | **MISS** (near) | claims.md:359 asserts *"no xemu-specific accelerator-selection code… none customizing tcg/kvm/hvf/whpx selection"* — the right neighbourhood, and honest, but never says xemu ships TCG-only, and `auto_features` is absent. |
| A3 | `pgraph_write` takes `pfifo.lock` + `pg->lock` | should | MISS (partial) | **MISS** ⬇ | **Regressed.** Zero lock claims anywhere; run 2 had PFIFO's mutex from the struct. |
| A4 | pushbuffer/FIFO vs PGRAPH are separate | should | HIT | **HIT** (partial) | `hwxbox-4`: per-engine files incl. `pfifo.c`, plus `pgraph/` as its own subdir; `hwxbox-5` on the backend registry. Structurally right. ⚠ Weaker than run 2 — `index.yml` has **no topic** for the PFIFO→PGRAPH seam, so a reader looking it up finds nothing. |
| A5 | nine `nv2a_lock_fifo` drain sites | deep | MISS | **MISS** | Not attempted. |
| B1 | two backends, runtime-selectable, not equivalent | **must** | HIT (partial) | **HIT** (partial) | `hwxbox-5`: `PGRAPHState` holds a runtime-selected `const PGRAPHRenderer *` from a static registry; `gl`/`vk`/`null`. Verified `pgraph.h:108-136`. *Not equivalent* clause untouched (behavioural, unreachable cold). |
| B2 | VK failure set ⊃ GL's | deep | MISS | **MISS** | Out of reach. |
| B3 | surface→texture gate → CPU readback | should | MISS (near) | **MISS** ⬇ | **Regressed.** Run 2 found the neighbourhood from `trace-events` density. Run 3 has no surface-path claim at all; `hwxbox-9` mentions "vk surface sync" only as a commit-log token. |
| B4 | gate rejects cubemaps / zeta→colour | deep | MISS | **MISS** | Not attempted. `hwxbox-9` says the word "cubemap" in a commit-subject list — not a claim about the gate. |
| C1 | `NV2A_PROF_*` set in `nv2a/debug.h` | should | MISS | ✅ **HIT** 🎯 | **Fixed.** `uixui` claims cite `debug.h:71-122` (the `NV2A_PROF_COUNTERS_XMAC` X-macro) and `debug.h:142,149-151`, note `NV2A_PROF__COUNT`, and state counters are populated by inline `nv2a_profile_inc_counter` calls from pgraph. Verified: 53 `NV2A_PROF_` entries in `debug.h`. **This was run 2's single highest-value miss.** |
| C2 | drawn in the shipped Video Debug panel | deep | HIT | **HIT** | `uixui-3`, `debug.cc:380-462`, ImPlot. Framed as shipped, not patched. |
| C3 | FPS is an integer over a 250 ms window | deep | MISS 🛑 | **MISS** | Still missed — but **less damningly**. Run 2 cited `profile.c:22-53`, a range *containing* both the window (line 27) and the integer division (line 36). Run 3 cites `profile.c:30-52`, which contains the division but **not** the 250 ms window. Narrower read, same omission: it names `increment_fps` with no resolution limit. |
| C4 | MSPF is not the frame period | deep | MISS | **MISS** | Calls it a "FPS/MSPF/draw-call perf overlay" and a "ring buffer of `NV2A_PROF_NUM_FRAMES=300`". No caveat. Not WRONG — it never asserts MSPF *is* the frame period. |
| D1 | `abaire/nxdk_pgraph_tests` is the accuracy instrument | **must** | MISS 🛑 | ⚠ **HIT (gifted)** | `abaire-1..4` profile it at R2 with correct provenance, and `gate-8` resolves the org fork's `parent` to abaire's copy. **But `BRIEF.md` named it as ground truth.** See § Validity 1. Credit the *resolution*; do not credit the *discovery*. |
| D2 | hardware golden corpus (~100 suites, ~5,600 PNGs, Xbox 1.0) | **must** | MISS | **MISS** (near) 🛑 | `abaire-3` says results are *"captured in a linked golden-results repo"* — existence noted, **repo never named, never opened, no counts, no hardware revision**. 🛑 **`abaire/xemu-nxdk_pgraph_tests_results` is in the output of `gh api users/abaire/repos --paginate`, which this map ran four times** (`abaire-4`, `-7`, `-15`, `-16` all cite it). Run 2 was "one hop short". **Run 3 is zero hops short — the answer was on screen and unread.** |
| D3 | `perceptualdiff` exit code | should | MISS | **MISS** | Absent. |
| D4 | tolerance <100 pixels, absolute | deep | MISS | **MISS** | Absent. |
| D5 | dead LPIPS backend | deep | MISS | **MISS** | Absent — and again available: the `#lpips~=0.1.4` comment sits in a sibling repo's `requirements.txt` (see § 3). |
| D6 | `nxdk_ntrc_dyndxt` traces the pushbuffer on hardware | should | MISS | **MISS** 🛑 | Absent. **Same failure as D2**: `nxdk_ntrc_dyndxt` is in the `gh api users/abaire/repos --paginate` output the map cites. `abaire-16`/`-17` selected *some* repos from that listing by description and dropped this one. |
| D7 | deployed over XBDM via `xbdm_gdb_bridge` | deep | MISS | **MISS** (partial) | `abaire-10..12` describe `xbdm_gdb_bridge` correctly as an XBDM↔GDB-RSP bridge, and `abaire-14` links the shared nv2a log format. The *tracer* half (D6) is absent, so the relation the row asserts is not made. |
| D8 | xemu's own PR CI is build-only | should | HIT 🎯 | **HIT** 🎯 | Reproduced, stronger. `cicd-1` (index: *"builds/packages/releases only; no test execution anywhere in `.github/workflows/`"*), corroborated by claims.md:97 (grep across **15** workflow YAMLs), :125 (zero `xemu-test` references), :201 (zero `nxdk`/`nxdk_pgraph_tests` references repo-wide). |

## The two headline questions

**A1 — would this map have prevented the wrong-thread error?** 🛑 **No, and less
so than run 2.** Run 2 got a reader to `nv2a_int.h` and a "dedicated worker
thread"; run 3 gets them to `qemu_main` and the BQL, which is the *UI* thread.
A reader trusting run 3's index would have no reason to suspect `MAIN` is not
the renderer.

**D8 — did it report the CI negative?** ✅ **Yes, and it is the map's best
work.** Four independent negative greps, promoted to an index topic in its own
right, with the `[docs]`-vs-`[source]` tension against `xemu-test`'s README
preserved rather than collapsed (`index.yml` topic *"Is xemu-test wired into
xemu's current CI"*, and `open-questions.md` § Contested). **This is the
`cartographer` hedge-preservation rule firing correctly, unprompted.**

## Off-key precision — 2 verified false assertions

`WRONG` on-key: **0**. Off-key, spot-checking 8 claims against source:

- 🛑 **`hwxbox-3` — "registers 5 SMBus peripherals at fixed addresses (smc 0x10,
  cx25871 0x45, fs454 0x6A, adm1032 0x4C, xcalibur 0x70)"**, citing
  `hw/xbox/xbox.c:299-313`. **False in every configuration.** The source is an
  `if/else` on the `video-encoder` machine property: `xcalibur` → smc +
  xcalibur (**2**); `conexant` → smc + cx25871 + adm1032 (**3**); `focus` → smc
  + fs454 + adm1032 (**3**). ⚠ **This is the worse of the two** — it is not a
  miscount, it is a **conditional flattened into a flat list**, which is exactly
  the fluency failure `EVALUATION.md` names as `surveyor`'s signature.
- ⚠ **`build-deps-1` — "`subprojects/` contains exactly 35 `.wrap` files"**,
  citing `ls subprojects/*.wrap`. Actual: **36**. Same class as run 2's 14-vs-15.
  ✅ Note run 3 got run 2's two errors *right*: **15** workflow YAMLs
  (claims.md:97) and **12** org repos (claims.md:415), both verified correct.

⇒ **The counting rule is not fixed; it moved.** Two runs, two off-by-ones, on
different `ls`-derived counts. **A rule that says "recount derived statistics"
is not producing a recount.**

## 3. What it found that the key does not contain

One item, verified, and it is **the most consequential thing either blind run
has produced**:

🎯 **`abaire/xemu-dev_pgraph_test_results` — a per-PR accuracy comparison
workflow.** The map named it in `open-questions.md` § Further leads (line 38),
correctly as a lead and not a claim, with a one-call resolver — **and did not
run it**. Running it took two `gh api` calls and produced
`the-xemu-cartographer/findings/F021`:

- `on: pull_request`, `runs-on: ubuntu-latest`, 4-way sharded, 45-min timeout —
  **no self-hosted runner, no secrets**, unlike the org's own harness.
- Compares against **hardware golden results** *and* **best-known xemu results**.
- ✅ **Live practice, not a template**: **11 xemu PRs** reference it
  (2025-02-28 → 2026-01-16, all `nv2a:`), and the repo holds **32** result
  branches named after xemu issues (`fix_559_…`, `fix_1852_…`).
- Its `requirements.txt` carries the **same commented-out `#lpips~=0.1.4`** the
  key's D5 row is about — so **D5's mechanism was two files from the map's own
  lead list.**

**Does it bear on why PGR1 is slow?** ❌ **No.** Accuracy diffs, not frame rate.
It bears on the *verdict*, not the mechanism — it is the third "already built"
finding in two sessions (`F019`, `F020`, `F021`).

## What this run says about the fixes

| Fix (2026-08-01) | Fired? | Evidence |
|---|---|---|
| **Cheap-band gate before hand-over** | ✅ **YES** | The run stopped and asked the human whether to spend ~2 min on 11 one-call resolvers. The human said run them → `record-gate-resolutions.yml`, 10 claims, which **superseded 4 index entries** (`build-overview-13`, `cicd-9`, `uixui-8`, `uifrontend-10`). **This is the clearest fix-fires-as-designed result in the evaluation so far.** |
| **One-call resolvers sit in the cheap band** | ✅ YES | `open-questions.md` § *Still open* — top 8 rows are all *"cheap — one git command"* / *"one `gh api`"*, and priced by the operation, not the category. |
| **`reach` carried into the fan-out** | ✅ YES | Every record carries `reach:`; `index.yml` has `reach_compiled: [inward, outward]`; `areas.md` has a pass log. ⚠ But the brief *mandated* `both` — untested whether the skill would have chosen it. |
| **Complete enumeration on a "what exists" direction** | ⚠ **MIXED** | Org enumeration: ✅ complete and verified (12/12). abaire enumeration: 🛑 **filtered, and the filter dropped two keyed rows** (D2, D6) from a listing it had already fetched. **It enumerated the org and sampled the person.** |
| **Late dispatch on referrals** | 🛑 **NO** | Six named leads in `open-questions.md` § Further leads, each with a concrete one-call resolver, **none dispatched** — including the one that turned out to be F021. The cheap-band gate fired for `not_determined:` entries and **did not fire for `unexpected:` leads.** |

## 🎯 The one change this run argues for

**The cheap-band gate must cover `unexpected:` free-text leads, not just
`not_determined:` entries.**

The gate fired, the human said yes, 11 resolvers ran, 4 index entries were
superseded — that worked. But the *same map*, in the *same file*, listed six
leads with one-call resolvers in a section the gate does not read, and **one of
them was the single most valuable thing in the run.**

⚠ Related but separate: the **abaire filter**. `abaire-16`/`-17` describe
themselves as selections from a `--paginate` listing. **A record that filters an
enumeration should say what it dropped and why**, or the enumeration rule is
satisfied in form and defeated in substance. D2 and D6 are both casualties of
that filter.

## Verdict

**6 / 15 / 0. Still a must-tier fail (2 of 4).** The two fixes aimed at named
misses landed (C1 outright, D1 with the brief's help). The **inward graphics
half regressed** — threads, locks and the surface path all went from "near" to
absent — which the `both` reach plausibly explains: the same budget bought
outward breadth at inward depth's expense. **`reach: both` is not free, and
`areas.md`'s pass log should be showing that cost.**
