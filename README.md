# premise · decision session viewer

Prototype viewer for **premise-carrying decision sessions**: an IDE sends decision
points to a decision pipeline, each decision carries premises (facts with sources),
the pipeline returns an advisory verdict with a confidence, and the bound verdict
becomes a citable premise for later decisions.

This is a working prototype with synthetic sample data modeled on compiler decisions
(register allocation, inlining, test selection). Single file, no build step, no
dependencies.

Live: https://clpi.github.io/premise-viewer/

## Screens

1. **Session list** (left): every session with its verdict posture at a glance:
   decision count, mean confidence, per-decision confidence sparkline, contested and
   override badges.
2. **Session detail** (center): a timeline of decision points. Expanding one splits it
   into the two directions the pipeline moves in:
   - **SENT** (`→`): the decision point, the question, and every premise with its
     provenance (source kind, locator, digest, collector, commit, confidence, quality).
   - **RETURNED** (`←`): the verdict, its confidence, model and ruleset, rationale,
     rejected alternatives, latency, and what it bound downstream.
3. **Premise inspector** (right): click any premise to see its full provenance, every
   decision that cites it directly, the transitive cascade through bound verdicts, and
   the retraction impact: what stops being supported and which gate artifacts return
   to unverified. **Simulate retraction** dims everything that would lose support.

## Data model

`PREMISES` is keyed by id and holds `{claim, src:{kind, locator, digest, collector,
commit, at}, quality, conf, note}`. Source kinds: `profile`, `ir`, `rule`, `trace`,
`test`, `diff`, `assumption`, `verdict`. A `verdict`-kind premise is a decision's
bound output, which is what makes a verdict citable as evidence later.

`DECISIONS` holds `{id, session, seq, kind, title, subject, question, premises[],
verdict:{advice, conf, status, model, ruleset, latency, rationale[], alts[], binds,
cites[]}, artifacts[]}`. `status` is `bound` / `advisory` / `contested` / `overridden`;
an `overridden` verdict is recorded but deliberately does not propagate.

The retraction cascade in `impact()` walks premise → citing decisions → each decision's
bound premise → decisions citing that, so blast radius crosses sessions.

To swap in real data, replace the `PREMISES` / `SESSIONS` / `DECISIONS` literals; the
renderer only reads those three shapes. Nothing else needs to change.
