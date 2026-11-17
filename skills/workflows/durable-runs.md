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

## Retry, continue, or stop

There is no continue capability. A parked or paused run is read, not resumed by a guessed call.

| Evidence | Decision | Call |
| --- | --- | --- |
| Succeeded and the trace has a receipt | Stop. Do not run it again. | `workflow.workflows.run_trace_get` |
| Failed, and no external write is ambiguous | Preview a retry from the first failed step. Completed steps stay preserved. | `workflow.workflows.retry` with `workflow_id`, `run_id`, and `dry_run` true, then live only under the paid-run consent rule |
| One subject failed at one node and the cause is fixed | Resume that node forward. `completed_effects_replayed` is false, so a sent message is not sent again. | `workflow.workflows.run_retry` with `subject_id`, `failed_event_id`, and `failed_node_id` taken from `workflow.workflows.run_trace_get`. Guessing them fails. The call refuses `durable_run_step_required` unless it is inside a claimed durable step. Say so instead of pretending it ran. |
| External write with a missing outcome | Stop. `workflow.workflows.retry` refuses `ambiguous_external_write`. | `workflow.workflows.run_get` and `workflow.workflows.run_events` |
| The user wants the live subject to end | Stop that subject. A step already at a provider is not recalled. | `workflow.workflows.run_cancel`. Completed, failed, and expired runs refuse `workflow_run_not_cancellable`. |

Read `workflow.workflows.runs_list` for `subject_id`. An empty list on a workflow that was never activated is empty, not an error. An empty trace is a different defect from a failed event. Do not use these calls on a worksheet run, a sequence enrollment, or a social post.

Metrics summarize finished runs. They never repair a run and never prove that one specific effect happened.
