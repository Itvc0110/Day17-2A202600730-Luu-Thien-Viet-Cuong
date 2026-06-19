# Reflection — Day 17 (≤ 200 words)

Answer briefly, in your own words. This is graded on reasoning, not length.

1. **The flywheel.** Day 13 emitted agent traces; today you turned them into an
   eval set and DPO pairs that Day 22 will train on. Which step in
   `traces → Bronze → datasets` would break most silently in production if you
   got it wrong — and how would you detect it?

2. **Decontamination.** Your run dropped 2 of 3 preference pairs because their
   prompts were in the eval set. What concretely goes wrong if you *skip* this
   step and train on those pairs? How would the lie show up in your metrics?

3. **Point-in-time.** The naive join leaked a future `lifetime_spend` into the
   training row. Describe one feature in a system you know that would be
   dangerous to join without an `ASOF`/point-in-time guard.

4. **Graph vs vector.** From `kg_demo.py`, name one question the knowledge graph
   answers well that flat chunk retrieval (`embed.py`) would struggle with, and
   one where the graph is overkill.

_Write your answers below._

The most silent failure is trace flattening: if child spans lose `trace_id`,
status, split, or token fields, the pipeline still runs but summaries, eval
rows, and DPO pairs become biased. I would detect it with span-count invariants,
per-trace rollups, and alerts on missing root outputs or unexpected status mix.

If decontamination is skipped, the model trains on prompts that are also in the
eval set. Offline metrics would look better because the model memorized grading
examples, but production accuracy would not improve on new questions.

A dangerous point-in-time example is credit scoring with "current total debt" or
"latest delinquency count". Joining the newest value into old training rows leaks
future behavior that was unavailable when the decision was made.

The graph answers multi-hop questions like "Where does a widget ship from?"
because it can traverse widget -> accessory -> Hanoi fulfillment center. A graph
is overkill for simple one-hop lookup such as "What is the widget return window?",
where vector or keyword retrieval is enough.
