# Answer key — a cold architectural map of xemu

**Written 2026-08-01, BEFORE the run.** Committed first so it cannot be adjusted
afterwards. This is test **D** (correctness against an oracle) from
`EVALUATION.md`, and it is the first run against the subject the skills were
actually built for: **onboarding to an unfamiliar codebase**, not compiling a
pile of findings.

## Why this is a valid oracle

The-xemu-cartographer spent twelve sessions establishing facts about xemu's
internals **by measurement and source-reading, not by impression**. Those facts
are hard-won, specific, falsifiable, and — critically — **were expensive to get
wrong**. A cold survey either finds them or it does not.

🛑 **The rows below are pointers, not content.** Each cites the finding that owns
it in `~/git_repos/the-xemu-cartographer`; that repo stays authoritative. This
file restates one falsifiable sentence per row so the run can be scored without
opening them, and nothing else. **The key is exactly as good as those findings.**

## Validity conditions — the run is void without these

1. **Blind.** The run must not read `the-xemu-cartographer` — not `/findings`,
   `/map`, `STATUS.md`, `questions/`, or the session logs. It reads xemu source
   and xemu's own docs/repos only.
2. **Read-only against the territory.** No `git` write of any kind in the
   surveyed clone (see `EVALUATION.md` § *Running it safely*).
3. **The direction is fixed before the run**, and is broad on purpose:

   > *"What is xemu's system architecture, and what related repositories exist
   > for testing, validation and performance profiling?"*

   ⚠ **Note what this direction does NOT ask.** It does not mention PGR1, frame
   rate, threads, or locks. **Scoring it against rows it was never pointed at is
   the unfair-standard failure**, which is what the tiering below exists to
   prevent.

## Tiers — what a fair grader expects at each depth

| Tier | Expectation |
|---|---|
| **T-must** | Any competent architecture overview must have this. Missing it is a **fail**, not a shortfall |
| **T-should** | A good map finds it. Missing it is a real gap; getting it is the map earning its keep |
| **T-deep** | Only a deep dive reaches it. **Getting it is a strong positive; missing it is not a mark against** |

**Score each row `HIT` / `MISS` / `WRONG`.** `WRONG` — asserted and false — is
the one that matters: a fluent map that is *wrong* is worse than no map, and a
`MISS` is honest if the map draws that area blank.

---

## A. Threads and the frame pipeline

| # | Falsifiable statement | Tier | Owner |
|---|---|---|---|
| A1 | xemu renders on **`pfifo_thread`**. The thread named `MAIN` is **not** the renderer | **T-must** | `F012` Claim 1 |
| A2 | The guest CPU runs in its own thread under **TCG** (QEMU's JIT), and **no shipped xemu build enables a hardware accelerator on any OS** — `-Dauto_features=disabled`, upstream commit `f2bfa95fae` (2021-05-19) | T-should | `F010` Claims 42/55/57 |
| A3 | Guest threads reach the GPU through **`pgraph_write`**, which acquires locks; the two that matter are **`pg->lock`** and **`pfifo.lock`** | T-should | `F012` Claim 9, `F013` Claim 1 |
| A4 | The pushbuffer/FIFO path and the PGRAPH path are **separate mechanisms** — a command stream fed by the guest, and a register/state machine acted on by the renderer thread | T-should | `F012` (architecture) |
| A5 | There are **nine `nv2a_lock_fifo` full-drain sites** in the codebase | **T-deep** | `F012` Claim 3 |

🎯 **A1 is the headline.** Session 004 of the source exercise measured the wrong
thread, concluded "the render thread is 3% busy", and killed a hypothesis on
that basis. It cost a session and the error survived undetected into three later
documents. **The single sharpest question this run answers: would this map have
prevented that?**

## B. Renderer backends and the surface path

| # | Falsifiable statement | Tier | Owner |
|---|---|---|---|
| B1 | xemu has **two renderer backends, OpenGL and Vulkan**, selectable at runtime, and they are **known not to be equivalent** | **T-must** | `F005` |
| B2 | Vulkan is the noisier of the two: across 11 matched runs its failure set **strictly contains** OpenGL's, failing ~3.2× as many tests, mostly hard crashes (`VK_ERROR_DEVICE_LOST`) rather than mis-rendering | T-deep | `F005` |
| B3 | A **surface-to-texture compatibility gate** exists; when it rejects a surface the path falls back to a **CPU readback per texture bind** | T-should | `F009` Claim 4 (**as a source reading — its role as PGR1's cause is REFUTED**) |
| B4 | That gate **rejects cubemaps** and **zeta→colour** conversions, with standing `FIXME`s, in **both** backends (`gl/surface.c`, `vk/texture.c`) | **T-deep** | `F009`, incl. the Vulkan cross-check |

## C. Instruments xemu already ships

| # | Falsifiable statement | Tier | Owner |
|---|---|---|---|
| C1 | xemu ships a **built-in NV2A profiling counter set** (`NV2A_PROF_*`, `hw/xbox/nv2a/debug.h`) covering things like surface-to-texture fallbacks and surface downloads | T-should | `F009` sub-answer 3 |
| C2 | Those counters are **drawn in a shipped UI panel** — the "Video Debug" window (`ui/xui/debug.cc`) — i.e. **no patched build is needed to read them** | **T-deep** | `F009`, `PLAN-F009` Phase 1 |
| C3 | The on-screen **FPS readout is an integer over a 250 ms window** — it cannot resolve a few fps | T-deep | `F012` Claim 7 |
| C4 | **MSPF is not the frame period.** It is a sub-frame interval that excludes the wait for flip, quantised to whole milliseconds, and stamped on a different thread from the one that computes it | **T-deep** | `F012` Claim 6 |

⚠ **C3 and C4 are the corpus's most expensive instrument lessons** — each one
retro-invalidated conclusions already written down. A map that names an
instrument without its resolution limit is doing the thing that caused them.

## D. The testing / validation / profiling ecosystem

**The direction asks for this explicitly, so the tiers are stricter here.**

| # | Falsifiable statement | Tier | Owner |
|---|---|---|---|
| D1 | **`abaire/nxdk_pgraph_tests`** — the homebrew NV2A test suite, built with the **nxdk** toolchain, is the ecosystem's accuracy instrument | **T-must** | `F001`, `F006` |
| D2 | **`xemu-nxdk_pgraph_tests_results`** holds the hardware **golden corpus** — ~100 suites, ~5,600 PNGs, captured on **Xbox 1.0 hardware** | **T-must** | `F006` |
| D3 | Comparison is by **`perceptualdiff`**, and pass/fail is its **exit code** | T-should | `F002` |
| D4 | The effective tolerance is **fewer than 100 differing pixels — an absolute count, not a fraction**, so strictness varies with resolution | **T-deep** | `F002` |
| D5 | `compare.py` carries an **LPIPS backend that never executes** (inverted `-no-lpips` flag; `lpips` commented out of `requirements.txt`), which also makes `--diff-threshold` inert | **T-deep** | `F002` |
| D6 | **`abaire/nxdk_ntrc_dyndxt` (`nvtrc`)** captures the **pushbuffer command stream** from a running commercial game **on real hardware**, plus per-draw register and surface dumps | T-should | `F004` |
| D7 | It is deployed over **XBDM**, the Xbox debug monitor, via **`xbdm_gdb_bridge`** — which injects it but is **not** where the tracer lives | **T-deep** | `F004` |
| D8 | **xemu's own PR CI is build-only** — no test invocation, no reference to `pgraph`, `nxdk_pgraph_tests` or `perceptualdiff` anywhere in its workflows | T-should | `F008` |

🎯 **D8 is the second headline.** It is a *negative* — the absence of a thing a
reader would assume is present. **Negatives are the hardest thing for a survey to
produce and the most valuable**, because nothing in the repo points at them.
Whether the map reports "no accuracy testing in CI" as a finding, rather than
silently omitting it, is the sharpest test of the surveyor's discipline in this
whole key.

---

## Scoring the run

**1. The key** — count `HIT` / `MISS` / `WRONG` per tier. Report three numbers,
not one. 🛑 **A single aggregate is how the first run was scored a success while
losing 45 of 49 open questions.**

**2. The two headline questions** — answer each yes/no with the evidence:
- **A1: would this map have prevented the wrong-thread error?**
- **D8: did it report the CI negative, or just not mention CI?**

**3. Anything it found that this key does not contain** — the point of the whole
exercise, and the part that must not be graded on vibes:
- **Verify against source before it counts as true.** Unverified goes in as a
  lead, exactly as the map's own rules require.
- **Then ask the only question that matters for the source exercise:** does it
  bear on why PGR1 is slow? Twelve sessions of mechanism-hunting have one live
  candidate. **The method that produced it was mapping the architecture before
  proposing a mechanism** — so this run is, incidentally, a second attempt at
  the thing that worked.

**4. Record the result as a row in `EVALUATION.md` § Scores so far** — including
if it goes fine. A fix is not validated by the session that wrote it.

## What this run still cannot test

- **Adjudication.** The key scores precision and coverage. It cannot score
  *"did it take the right side where the corpus held two"*, because a fresh
  codebase holds no contested positions — the code is simply what it is.
  That remains run 2 on a domain the human knows cold.
- **The unmeasured majority.** This key covers the **graphics path and the test
  ecosystem only**, because that is the only territory the source exercise
  explored. Audio, input, UI, save states, and the QEMU integration have **no
  oracle here** — the map may be excellent or fabricated in those areas and this
  file cannot tell. **Do not read a good score as a verdict on the whole map.**
