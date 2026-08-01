---
name: visualiser
description: Render a compiled map as diagrams a human can read at a glance — constellation of subjects and relations, architecture, coverage-and-resolution, or status — with inferred links drawn differently from sourced ones and unmapped areas drawn as blank. Use when a map or set of findings has become too large to parse by reading, when asked to visualise, diagram or show the map, or when deciding where to survey next.
maturity: draft
---

# Visualiser

**Speaks map format v1.** Reads the **index**. Renders for a **human**.

Writes no claims and changes no status. If you find something wrong with the
map while rendering it, say so — do not fix it here; that is `cartographer`.

## Why this exists

A map that has to be *consulted* through an assistant is not readable. The
symptom is a human asking what their own map says. Your output should mean they
no longer have to ask.

## 1. Render from the index, never from the claims

🛑 **Read the index.** It is the only part of the map that knows what is
current. Rendering from the underlying claims will draw superseded findings
beautifully and confidently, which is the worst failure available here.

If the map declares a format version other than v1, stop and say so.

## 2. Ask what the human is looking at it for

One question before drawing: **what are you trying to see?** The answer picks
the view. If they do not know, offer the four:

| View | Answers | Shape |
|---|---|---|
| **Constellation** | What is here and how does it relate? | Node graph of subjects and relations |
| **Architecture** | How is this thing put together internally? | Component or sequence diagram |
| **Coverage** | Where is the map thin, and where should I survey next? | The territory shaded by resolution |
| **Status** | What is settled, contested, or stale? | Board or table by index status |

**Coverage is the one people do not think to ask for and most often need** — it
is the only view that shows the shape of what is *not* known, and it cannot be
expressed as a list without becoming another wall of text.

## 3. 🛑 A diagram is a claim

Diagrams are far more persuasive than prose, so an unsourced one does more
damage. Three rules, without exception:

1. **Draw grounding.** `[source]` relations solid; `[inferred]` relations
   **dashed**; `[docs]` dotted. A guessed relation must never look verified.
   Put the key on the diagram itself.
2. **Blank is drawn as blank.** An unmapped area gets its boundary drawn and its
   inside left empty, labelled as unmapped. **Omitting it reads as "nothing
   there", which is a claim nobody made.**
3. **Mark contested topics as contested** — both sides shown, neither presented
   as the answer.

Every diagram carries, visibly: **the direction it serves**, the **date**, and a
**key**. A diagram found later without those is indistinguishable from a
confident guess.

Show resolution wherever it can be shown — opacity, border weight, or an
explicit R0–R3 label. An R1 sketch drawn identically to an R3 survey is the
fluent unsourced map arriving by the back door.

## 4. Succinct explanation beside every diagram

Three to five lines, and they must include:

- **what the diagram is claiming**, in one sentence;
- **what is missing from it** — the blank areas, in words as well as shape;
- **what to do next**, if the picture implies something (usually: which thin
  area to survey).

Never let the prose assert anything the index does not contain. If you want to
say it and the map does not support it, that is a finding for `cartographer`,
not a caption.

## 5. Output

Default to **mermaid in markdown** — it renders in most viewers and diffs in git.

Offer a **published artifact** when the map is large, when several views are
wanted at once, or when the human wants to come back to it: mermaid renders
natively there, it gets a URL, and redeploying the same file updates it in
place — which suits a map whose resolution increases over time.

Keep diagrams readable rather than complete. **A diagram that shows everything
shows nothing** — if a view needs more than roughly a dozen nodes, split it by
area and say how the pieces join.

🛑 **Write only to the map location** (or to an artifact). **Never edit the
domain's own documents or source** — do not drop diagrams into a README, a status
page or a findings file because they would look good there. **Renders scatter
more easily than any other output**, and a diagram pasted into a document nobody
re-renders becomes a confident stale picture the moment the index moves. Offer
the human the file; let them place it.

## 6. Report what you could not draw

Finish by naming, in a line or two:

- topics the index marks **contested** — they resist a single picture, which is
  the point;
- **`live` claims whose `last_checked` is old**, because a diagram makes a stale
  number look freshly true;
- 🛑 **entries the index marks `unverified`, or any figure it gives without an
  `n`** — the same hazard as staleness and a worse one, because a stale number
  was at least true once. **Either draw it with the unverified marking the index
  gave it, or leave it out and say you did.** Never render it as a bare number;
  a diagram strips every qualifier the index was careful to attach;
- anything in the map that could not be rendered honestly, and why.

**Do not fix the map to make the picture nicer.** A map that renders awkwardly
is telling you something true about the territory.
