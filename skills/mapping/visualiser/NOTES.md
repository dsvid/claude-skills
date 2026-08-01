# visualiser — notes

Append friction as it happens: what was ambiguous, what had to be worked around,
what the human corrected. One line each. This is the evidence for promoting the
maturity rung.

## Open questions, written before the first run

- **Can mermaid express grounding legibly?** Solid / dashed / dotted for
  `[source]` / `[inferred]` / `[docs]` is the whole honesty mechanism, and it
  rests on an untested assumption that the distinction survives rendering at
  realistic node counts. **If it does not read at a glance, the honesty rules
  are decorative and the skill's central claim fails.**
- **Can the coverage view be drawn at all?** Shading territory by resolution is
  the view argued to be most valuable and least requested — and it is the least
  natural fit for a graph-oriented diagram language. It may need a different
  representation entirely (a table, a treemap, a grid).
- **Is "blank drawn as blank" achievable in a graph?** Easy on a physical map,
  awkward when there is no canvas. An explicit "unmapped" node may be the best
  available, and may look like a subject rather than an absence.
- **Does asking "what are you trying to see?" help or annoy?** One question
  before drawing. If the answer is nearly always the same view, drop the
  question and default to it.
- **Does the dozen-node limit hold?** A guess. Find where legibility actually
  breaks and record the number.
- **Does anyone want the artifact, or is markdown enough?** The artifact route
  has a real advantage for a map whose resolution grows — same URL, updated in
  place — but it is more ceremony per render.

## Watch for

- **Rendering something the index does not contain.** The most likely way this
  skill does damage: a caption or an edge that reads as established when it was
  inferred while drawing. Every element should be traceable to an index entry.
- **The map being tidied to make the picture nicer.** A map that renders
  awkwardly is usually telling the truth about the territory.

## Friction log

<!-- newest first; date each entry -->

### 2026-08-01 — 🛑 §5's DEFAULT IS FALSIFIED. "Mermaid in markdown renders in most viewers" is wrong where it matters

**Held for the step-3 skills review** (the human deferred the fix mid-run to
preserve s011's ordering — review all three skills *after* the consolidation
pass, not during a run). **Do not treat this as fixed.**

**What happened.** The first run wrote `views.md`, per §5's default. The human
opened it in **VSCode's markdown preview** — the single most likely viewer for a
skill that says "it diffs in git" — and the diagram was **too small to read and
could not be navigated.** Their words: *"the mermaid diagram is really small — is
there a better way for me to navigate it?"*

**The mechanism, so the review does not have to rediscover it:** mermaid stamps a
`max-width` on the SVG it generates, matching its container. A wide diagram
therefore **scales down to fit rather than scrolling.** The fix is
`svg { max-width: none }` plus an `overflow: auto` container of its own — after
which it needs pan and zoom to be usable at all. **Markdown previewers give you
none of that**, and the skill cannot inject CSS into them.

⇒ **§5 currently recommends, as its default, the one route that cannot produce a
navigable diagram.** The recommendation is not merely incomplete; the stated
reason for it ("renders in most viewers") is false for any view wider than a
preview pane. **Every view this run produced was wider than a preview pane.**

**What the run actually did** (per-instance output, correctly placed in the map
location — not a skill change): wrote a hand-built `map/views.html` with zoom
steps, drag-to-pan, fullscreen and auto-fit-on-render, and published it as an
artifact. **The artifact route worked well** and answers the last open question
above (*"does anyone want the artifact, or is markdown enough?"*) — **markdown
was not enough**, on the very first run, unprompted.

**The three routes as they actually stand, for §5 to be rewritten around:**

| Route | Navigable | Diffs in git | Needs |
|---|---|---|---|
| mermaid in markdown | ❌ shrinks, no pan/zoom | ✅ | nothing |
| standalone HTML per diagram | ✅ | ✅ (small files) | mermaid from a CDN, or vendored |
| published artifact | ✅ renders natively | ❌ | the Artifact tool |

**The open design question the review must settle** — and the reason this was
deferred rather than patched: producing navigable HTML per run means **every run
rewrites the same viewer boilerplate.** A `render.py` in the skill directory
would fix that, but **the mapping family has no scripts at all** — three skills,
six files, all prose. Adding one is a change to what this family *is* (portable,
language-agnostic instructions), not just a change to `visualiser`.
🛑 **That is a family-level call and belongs in the step-3 review, alongside
`cartographer`'s two outstanding contract changes.**

### 2026-08-01 — FIRST RUN. Coverage renders well; the four-view menu does not hold; the constellation prediction is UNTESTED, not confirmed

**Setup:** cold session in `the-xemu-cartographer`, run against `/map/index.md`
(492 lines, 24 records / 296 claims behind it). Read `index.md`, `map/README.md`
and `areas.md`; **opened no survey record and no finding.** Output:
`/map/views.md`, two diagrams.

**Scored against the pre-registration:**

- ✅ **The "index only" contract held for coverage.** The coverage view drew
  fully — R1/R2/R3 shading, seven blank areas with boundaries and empty
  interiors, contested entries marked — with nothing opened below the index.
- ⚠ **But "the index" is really `index.md` + `areas.md`.** The coverage view is
  close to a rendering of `areas.md` alone: it is the file that carries
  resolution levels, the thin-area ranking and the blank table. **The contract
  should say so.** A cartographer that emitted only `index.md` would leave the
  most-argued-for view undrawable.
- 🔎 **The constellation prediction was NOT tested.** The human picked coverage
  and architecture; no constellation was drawn. What was observable while
  reading: the index *does* carry supersession relations explicitly, in prose
  inside entries (`supersedes: f012#14`, `contested_between:`), so a
  **supersession chain would render** — but no other relation kind appears at
  all. Best current estimate is **"thin but honest", not "unbuildable"**, and it
  is an estimate. **Do not mark the prediction resolved.**

**New friction, unpredicted:**

- 🛑 **The four-view menu implies all four views are always available. They are
  not.** Views orthogonal to the direction the map was *compiled against* are
  badly under-supported. This map's direction is about **evidence currency**, so
  the architecture view had to be assembled from fragments of the instruments
  section and came out at R1–R2 with an explicitly unmapped interior — while the
  coverage view came out at full resolution. **§2's table should warn that a view
  is only as good as its alignment with the map's direction**, and the skill
  should say this *when offering the menu*, not after drawing.
- **Asking the question was worth it, and the answer was two views, not one.**
  §2 says "one question… the answer picks the view", singular. Multi-select was
  the right shape. Cost: one question, and it changed the output materially.
- **Mermaid expresses grounding legibly at ~12 nodes** — dashed vs solid edges
  read at a glance, and `stroke-dasharray` on a transparent-fill node is a
  convincing "blank": it looks like an absence rather than a subject. First open
  question above is answered **yes**, at this node count.
- **The index frequently does not state a claim's grounding tag** — it points at
  `<record>#<n>`, and the tag lives in the record. So §3's "draw grounding" rule
  is only partly executable from the index. Worked around by widening the key:
  *dashed = inferred **or grounding not stated in the index***. **That is a
  fudge, and it belongs to the cartographer** — the index should carry the tag
  it points at.

### 2026-08-01 — PRE-RUN PREDICTION: the index may not carry enough RELATIONS to draw a constellation

⚠ **Written before the first run, so the run can score it.** This is a
prediction, **not a finding.**

**Setup:** the first real `cartographer` compile (24 records, 296 claims →
`/map/index.md`, in `the-xemu-cartographer`) is about to be rendered. The
visualiser reads **the index only**, by contract.

**The prediction:** it will not be enough for the *constellation* view.

- The index that compile produced is keyed by **topic → current claim**, and
  carries **status, resolution, volatility, supersessions and contested pairs**.
- What it carries almost none of is **claim-to-claim relations**. Those live in
  the survey records' `relations:` blocks (`refutes`, `depends on`, `measures`,
  `supersedes`) — **which the visualiser is not allowed to read.**
- So the coverage / resolution / status views should render fine. **The
  constellation should come out thin or unbuildable.**

**Two ways that can go, and they have different fixes:**

| Outcome | What it means | Fix belongs to |
|---|---|---|
| Renders thin but honest | Acceptable — the map genuinely has few recorded relations | nobody; record it |
| Cannot be drawn without opening the records | **The index is under-specified**, and the "index only" contract is either wrong or is a constraint the cartographer must be told to satisfy | 🛑 **`cartographer`** — it should carry relations into the index |

🛑 **The failure to avoid: quietly widening the visualiser's read permission to
the records.** That would make the picture nicer and destroy the only structural
guarantee this skill has — that a diagram cannot assert more than the index does.
**If the index is insufficient, say so; do not route around it.**

**Falsifier:** if the constellation renders usefully from the index alone, this
prediction is wrong and the current cartographer output format is sufficient.
Say so plainly.

**Related:** `skills/mapping/NOTES.md` — this is a boundary question between two
skills, not a rendering bug.
