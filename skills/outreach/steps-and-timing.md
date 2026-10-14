# Steps and timing

Authorable steps are the ids `sequence.sequences.step_options` returns. That list is `canonicalSequencePaletteOptionIds()` in `packages/types/src/canonicalSequences.ts` (`CANONICAL_SEQUENCE_STEP_KINDS`, `CANONICAL_SEQUENCE_ACTIONS`, `CANONICAL_SEQUENCE_LINKEDIN_ACTIONS`, user-pickable nodes only). Read the schema for the exact shape, including whether a delay is its own step (`canonicalSequenceStepDelaySecondsSchema`). Do not invent a step the palette does not list. A LinkedIn message never sends an invite. To reach someone who is not connected, put a connection request step first when the palette offers one, then the message steps; a message waits until the request is accepted. Leave the connection request note empty by default: a blank invitation is accepted more often and the message after acceptance does the talking. Write a note only when the user asks for one. Skip the connection request only when the audience is already connected (first-degree) or an earlier step waits for the person to connect.

Typical cadence: three to five touches over two to three weeks, two to five days apart, mixing channels when both are available. An email follow-up reads best as a reply in the same thread. Put the strongest message first, and make the last touch an easy way to close the loop.

A reply, an unsubscribe or a failed send ends a lead's run on its own, so do not add a condition that only checks reply state. Sequence conditions may branch on worksheet-row variables, such as `fit_score`, `title`, or `first_name`, using `expression` plus both `true_step_id` and `false_step_id`. Keep reply as stop. Never refuse whole build over one step.

Build the whole sequence in one pass when you know every step. Add, edit, remove or reorder a single step when the user asks for a narrow change, and leave the rest alone. Read the real step settings before writing them rather than guessing field names or time units.

After every change, validate, fix every real error and validate again. Read the saved sequence back before saying it is ready.

Delivery is paced by sending hours, per-account cooldowns and daily caps, so a start is never an instant blast. Enrichment for enrolled leads runs only while the sequence is active.
