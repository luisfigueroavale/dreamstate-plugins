# Failed runs and unknown outcomes

Start from the run history, then open the trace of the one run in question. The trace is the truth for each node: its state, attempt, output, receipt and error.

Take identifiers from what you inspected, never from memory or guesswork. An empty run list or a trace without a failed node means there is nothing to retry yet.

How to recover:

- Succeeded with a receipt: never run it again.
- Failed before any effect: retry that node.
- Effect definitely not applied: retry with the same idempotency identity.
- A preview, dry run or read that timed out applied nothing: retry the same call.
- Timeout, lost connection or missing receipt: the outcome is unknown. Check the outside system or the trace first. When the read-back shows the write absent, retry it with the same idempotency key: the server replays a key it has seen and never applies it twice.
- Paused or checkpointed: continue from the checkpoint, not from the start.
- An effect that cannot safely repeat and whose outcome is still unknown: stop and tell the user.

A retry targets the failed node, not the whole run. Completed nodes and earlier effects never repeat. An accepted retry is not a finished run; follow it to its final result.

Metrics summarize finished runs. They never repair a run and never prove that one specific effect happened.
