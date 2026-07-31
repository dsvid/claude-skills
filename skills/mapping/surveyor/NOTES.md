# surveyor — notes

Append friction as it happens: what was ambiguous, what had to be worked around,
what the human corrected. One line each. This is the evidence for promoting the
maturity rung.

## Open questions, written before the first run

Pre-registered so a first run can answer them rather than just feel fine.

- **Does refusing to start without a direction help or obstruct?** It is the
  stopping rule and the fan-out instruction, so it looks load-bearing — but a
  user who "just wants a look around" may find it pedantic. Watch whether the
  direction that gets written is real or invented to satisfy the gate.
- **Is the YAML record too heavy to actually fill in?** The likeliest failure of
  this skill. If records come back thin, half-filled, or with `provenance` doing
  the work of four fields, the schema is wrong, not the surveyor.
- **Will the tiers get used, or will everything be recorded as T2?** T0 and T3
  in particular. If nothing is ever T0, that probably means unprovenanced claims
  are being quietly dropped or dressed up rather than marked.
- **What is the sub-agent fan-out threshold?** "More than a couple of subjects"
  is a guess. Below some N the coordination costs more than it saves.
- **Does `unexpected` get filled?** It exists because the by-product often beats
  the target. An empty `unexpected` on every record means fan-out is flattening
  exactly what it was meant to preserve.
- **Does step 2 actually fire?** "Ask what existing material can still answer
  this" and "has someone already answered this" are the two cheapest steps and
  the two most likely to be skipped in favour of gathering.

## Friction log

<!-- newest first; date each entry -->
