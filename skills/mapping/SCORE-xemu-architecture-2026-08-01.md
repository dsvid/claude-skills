# Score — the blind xemu architecture run, 2026-08-01

Scored 2026-08-01 (source exercise session 014) against
`ORACLE-xemu-architecture.md`, which was **written and committed before the
run**. This file is the detail; `EVALUATION.md` § *Scores so far* carries the
one-line row.

- **Map scored:** `the-xemu-cartographer/map_fresh/` — 12 records, `INDEX.md`,
  `CLAIMS.md`, `AREAS.md`, `OPEN_QUESTIONS.md`. Byte-identical to the human's
  `~/git_repos/xemu-map-temp/`, confirmed by `diff -rq` (exit 0).
- **Subject:** `~/git_repos/xemu` @ **`cbffb57d08`**. Every verification below
  was run against that SHA via `git show cbffb57d08:<path>`, not against the
  working tree — the clone has since been moved to `bench/pr2941-test`
  (`28100e600c`), which is a *different* tree.
- **Validity conditions held:** the run was blind (no reference to the source
  exercise anywhere in the output), and read-only (checked at s013 close:
  clean `git status`, no new branches, reflog shows only the human's own
  `checkout`/`pull`).

⚠ **Row-count correction.** Both this key's own preamble and `EVALUATION.md`
describe it as *"17 tiered rows"*. It has **21** (A1–A5, B1–B4, C1–C4, D1–D8).
Recounted mechanically before scoring. The oracle file is left unedited —
it is pre-registered — but the number it states about itself is wrong.

---

## 1. The key — three numbers, never one

| Tier | HIT | MISS | WRONG | Rows |
|---|---|---|---|---|
| **T-must** | **1** | **3** | 0 | A1, B1, D1, D2 |
| **T-should** | **2** | **6** | 0 | A2, A3, A4, B3, C1, D3, D6, D8 |
| **T-deep** | **1** | **8** | 0 | A5, B2, B4, C2, C3, C4, D4, D5, D7 |
| **Total** | **4** | **17** | **0** | 21 |

🛑 **Three of four T-must rows missed. By the key's own rule — *"missing it is a
fail, not a shortfall"* — this run fails at the must tier.**

✅ **Zero WRONG on every keyed row.** Nothing in the key's territory was asserted
and false. Where the map did not know, it drew the area blank and said so —
`INDEX.md` marks four topics **Thin** and routes each to `OPEN_QUESTIONS.md`.
That is the failure mode the whole method exists to prevent, and it did not
occur.

### Per row

| # | Tier | Score | Why |
|---|---|---|---|
| A1 renders on `pfifo_thread`, `MAIN` is not the renderer | **must** | **MISS** (near) | `nv2a-gpu-4` finds PFIFO's `QemuThread`/`QemuCond`/`halt` in `nv2a_int.h:96-106` and says FIFO processing runs on *"a dedicated worker thread rather than purely the main/vCPU thread"*. It never names `pfifo_thread`, never says **rendering** happens there, and `INDEX.md`:52 files the PFIFO→PGRAPH dispatch path as **thin**. Verified in source: `pfifo.c:452 void *pfifo_thread()` → `:464 pgraph_process_pending(d)`. **The map stopped one file short.** |
| A2 TCG, no accelerator in any shipped build | should | **MISS** | No claim about TCG/KVM/WHPX/`auto_features` anywhere in the map. The only `TCG` strings are `tests/tcg` directory listings. |
| A3 `pgraph_write` takes `pfifo.lock` + `pg->lock` | should | **MISS** (partial) | `nv2a-gpu-3` notes PFIFO has "its own mutex", from a struct listing. No lock-acquisition site, no `pg->lock`, no `pgraph_write`. Verified: `pgraph.c:83` `pgraph_write` → `:90` `pfifo.lock` → `:91` `pg->lock`. |
| A4 pushbuffer/FIFO and PGRAPH are separate mechanisms | should | **HIT** | `nv2a-gpu-1` (per-register-block files incl. `pfifo.c`) + `nv2a-gpu-5` (`pgraph/` as its own source set with gl/vk/null backends), and `INDEX.md` states the split explicitly. Structurally correct, and it flags the seam between them as the unknown. |
| A5 nine `nv2a_lock_fifo` full-drain sites | deep | **MISS** | Not attempted. Verified independently: 9 occurrences in `nv2a.c`. |
| B1 two backends, runtime-selectable, not equivalent | **must** | **HIT** (partial) | Found **three** (`gl`, `vk`, `null`) plus RenderDoc — more than the key. Runtime selection is flagged thin, and the *not equivalent* clause is untouched. That clause is behavioural and unreachable by reading; scored HIT on what a cold read can reach. |
| B2 Vulkan's failure set strictly contains OpenGL's | deep | **MISS** | Requires 11 matched runs. Out of reach. |
| B3 surface-to-texture gate → CPU readback fallback | should | **MISS** (near) | `nv2a-gpu-7` reads `trace-events` and concludes surface lifecycle (create/hit/match/evict/upload/**download**) is *"a heavily-traced area"* — it found the right neighbourhood from trace density alone, without the mechanism. |
| B4 gate rejects cubemaps and zeta→colour, both backends | deep | **MISS** | Not attempted. |
| C1 `NV2A_PROF_*` counter set in `hw/xbox/nv2a/debug.h` | should | **MISS** | Found `profile.c` and `NV2A_PROF_NUM_FRAMES` but never opened `debug.h`, which holds **53** `NV2A_PROF` entries including `NV2A_PROF_SURF_TO_TEX_FALLBACK` and `NV2A_PROF_SURF_DOWNLOAD` (verified, `debug.h:73-111`). The single highest-value instrument in the corpus, missed by one header. |
| C2 counters drawn in the shipped Video Debug panel | deep | **HIT** | `profiling-tooling-6`: `ui/xui/debug.cc:384-405`, ImPlot scrolling FPS plot reading `g_nv2a_stats`. Correctly framed as shipped, not patched. |
| C3 FPS readout is an integer over a 250 ms window | deep | **MISS** 🛑 | `profiling-tooling-5` cites **`profile.c` lines 22-53** — the window is at line 27 (`fps_update_interval = 250000`) and the integer division at line 35. **The map had the lines open and reported the instrument without its resolution limit.** This is verbatim the failure the key warns about at § C. |
| C4 MSPF is not the frame period | deep | **MISS** | Same claim calls `frame_working.mspf` *"per-frame timing"*. Verified: `mspf = (now - last_flip_time)/1000` — whole ms, excludes the wait for flip. Not scored WRONG (it does not assert equality) but it is the loose framing that produced the corpus's worst instrument error. |
| D1 `abaire/nxdk_pgraph_tests` is the accuracy instrument | **must** | **MISS** 🛑 | Absent. See § 3 — it was **one API call away**. |
| D2 hardware golden corpus (~100 suites, ~5,600 PNGs, Xbox 1.0) | **must** | **MISS** | Absent. |
| D3 comparison by `perceptualdiff` exit code | should | **MISS** | Absent from the map — though `perceptualdiff` turns up in the follow-up verification of the map's *own* lead (§ 3). |
| D4 tolerance is <100 differing pixels, absolute | deep | **MISS** | Absent. |
| D5 `compare.py`'s LPIPS backend never executes | deep | **MISS** | Absent. |
| D6 `nxdk_ntrc_dyndxt` traces the pushbuffer on real hardware | should | **MISS** | Absent. |
| D7 deployed over XBDM via `xbdm_gdb_bridge` | deep | **MISS** | Absent. |
| D8 xemu's own PR CI is build-only | should | **HIT** 🎯 | The run's strongest result — see § 2. |

---

## 2. The two headline questions

### A1 — would this map have prevented the wrong-thread error? **No.**

It would not have *caused* it either, which is the weaker but real consolation.
A reader who had this map before session 004 would know a dedicated PFIFO worker
thread exists and that the PFIFO→PGRAPH seam is **unmapped** — flagged thin in
the index, with the resolver named in `OPEN_QUESTIONS.md`. They would not know
which thread renders. Profiling "the render thread" would still have been a
guess; the map's contribution is that it marks the guess as a guess.

**The gap is one hop.** The map read `nv2a_int.h` for the thread's *declaration*
and never opened `pfifo.c` for its *body*, where line 452 names `pfifo_thread`
and line 464 calls `pgraph_process_pending`. A "when a struct declares a thread,
read its entry function" step converts this row from MISS to HIT — the same
shape of gap as C1 (`debug.h`) and D1 (the org listing).

### D8 — did it report the CI negative? **Yes, emphatically.** 🎯

Not merely mentioned — **promoted**. `INDEX.md` opens the topic with a bare
**"No."**, calls it *"the most load-bearing finding in the map for the
testing/validation half of the direction"*, corroborates it along two
independent routes (`xbox-swizzle-test-4`: no CI reference to the one xbox unit
test; `qemu-inherited-test-suites-1`: zero xbox/nv2a/mcpx matches in the
inherited test trees), and raises it to **Corrections owed #1** marked
*verified, not a lead*. It also bounds itself: *"Not checked: workflows outside
the five named."*

This independently reproduces `F008` Claim 1 from a cold start, and it is the
clearest evidence yet that the method can produce a **negative** — the thing the
key called hardest to get and most valuable.

⚠ One off-key count error inside the win: the map says `.github/workflows/*.yml`
is **14 files**; at `cbffb57d08` it is **15**. `F008` Claim 1 says 15 and is
right.

---

## 3. What it found that the key does not contain

**Verified against source/API before counting, per the key's rule.** Full
write-up of the consequential one is `the-xemu-cartographer/findings/F019`.

| Found | Verified? | Notes |
|---|---|---|
| **`xemu-project/xemu-test` + `xemu-test-agent`** — an org-level automated test system | ✅ **VERIFIED and consequential** | The map's own weakest-flagged entry (R1, `[docs]`, "most likely to be wrong if quoted"). It is **right**, and it is a genuine hole in 13 sessions of the source exercise — `grep -rn "xemu-test"` across `findings/`, `STATUS.md`, `map/`, `questions/` returns **nothing**. See F019. |
| `hw/xbox/chihiro.c` disabled in the default build | ✅ verified | `hw/xbox/meson.build:4-5`, both lines commented out. |
| `tests/xbox/swizzle/` is Makefile-only, not in Meson, not in CI | ✅ verified | `tests/xbox/` contains exactly `swizzle/Makefile` + `swizzle-test.c`; no Meson reference. Strengthens D8. |
| MCPX APU carries a **DSP JIT** alongside an interpreter | ✅ verified | `hw/xbox/mcpx/apu/dsp/dsp_jit.c` and `dsp/interp/dsp_cpu.c` both exist. |
| `ui/xemu.c` registers its own `DISPLAY_TYPE_XEMU` while QEMU's `sdl2*.c` stays in the tree | ✅ verified | `ui/xemu.c:1187`; `ui/meson.build:165-169`. Reachability correctly left open. |
| 14→**15** chained workflows; release fan-out dispatches into `xemu-website`; Windows builds in a sha-pinned GHCR toolchain container | ⚠ mostly verified, count wrong | The container detail independently reproduces `F010` Claims 41-43. |
| Org has "12 repos" | ⚠ **wrong** | 13 public repos returned by the API. It was `[docs]`/T2 from a search snapshot and hedged as such. |

**Does any of it bear on why PGR1 is slow?** **Essentially no** — nothing new on
threads, locks, or the surface path. The one operational consequence is F019:
an existing containerised harness that already runs the accuracy suite against
xemu builds, which bears on the *rig* half of the source exercise's verdict and
on its Q9, not on the mechanism.

🎯 **The sharpest single result in this run is a process one.** The map's
**least** confident entry — hedged, R1, flagged twice, with its own resolver
written down (*"Direct fetch of the org repositories page"*) — was the one that
paid. Executing that resolver took **one `gh api` call**, and the returned
listing contains **`nxdk_pgraph_tests`**: row **D1**, a T-must miss, sitting in
the enumeration the map explicitly recorded as incomplete.

⇒ **The map did not fail D1/D2 by fabricating or by omission. It failed by
stopping at the boundary of a gap it had correctly identified and priced.**
That is a far better failure than a fluent wrong answer, and it points at a
concrete skill change rather than a vague "try harder": **`surveyor` should
execute a `resolved_by` when it is one cheap, non-mutating call — or
`cartographer` should promote such items above the compile.** Written up in
`surveyor/NOTES.md`.

---

## 4. What this score does not license

- **The key covers the graphics path and the test ecosystem only.** The map's
  xbox-platform, mcpx-apu, mcpx-nvnet, xemu-ui, build-versioning and
  release-pipeline areas have **no oracle**. Spot-verification of six of their
  claims (§ 3) found five right and one count wrong — encouraging, not a verdict.
- **4/21 is not "the skills are 19% correct."** The direction never mentioned
  threads, locks, PGR1 or frame rate; area B's and D's deep rows need runs the
  survey could not perform. The load-bearing number is the tier split, and the
  fact that **the must tier failed 3 of 4** while **WRONG stayed at 0**.
- **Adjudication is still unscored.** A cold codebase holds no contested
  positions. That remains a separate run.
