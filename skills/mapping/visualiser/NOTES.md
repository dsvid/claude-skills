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
