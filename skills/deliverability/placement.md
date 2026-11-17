# Placement, evidence, and monitoring

## Evidence loop

`senders.deliverability.evidence_list` returns rows from the last 400 days. Optional `kind` is `workspace_copy`, `seed_placement`, `copy_score`, or `dns_check`. Pass a row cap only when `tools_get` shows that field. Read `placement_test_available` on the list result.

`senders.deliverability.evidence_compare` takes `left_id` and `right_id` of the same kind. It diffs by `receiver_provider` with sample sizes and per-folder `rate_left`, `rate_right`, and `delta_rate`. A null rate means an empty sample.

Loop: hypothesis, cheapest check, store or read the result, compare with the earlier row, then propose one approval-gated action. Do not buy a second test because the first report is incomplete. Missing seeds are unknown, not spam.

`senders.copy.spam_score` scores 0 to 100 from trigger phrases, links, image ratio, caps, exclamation runs, and a missing unsubscribe line when `bulk` is true. It does not send.

## Workspace copy versus seed probe

| Question | Capability | Sends? |
| --- | --- | --- |
| Which of our connected mailboxes would receive this copy, and in which folder? | Preview, then `senders.deliverability.workspace_test_run` | Preview no. Run yes. |
| How do seed providers folder one mailbox's probe? | `senders.mailboxes.placement_test` | Yes |
| Did two stored runs differ? | `senders.deliverability.evidence_compare` | No |

Workspace run returns Gmail labels or Microsoft `inferenceClassification` after a short poll. Seed run returns `by_provider` counts for `inbox`, `spam`, `promotions`, `other`, and `missing`, plus `status`, `purpose` `placement_probe`, `fingerprint`, and `advisory`.

## Monitoring cost and approval

`senders.mailboxes.deliverability_connect` previews or connects selected mailboxes. Catalog text: managed mode is 200 credits per inbox-month, 2 placement tests per inbox per cycle included, extra managed placement test 132 credits. Own-key prices are null. Preview first. A live connect follows the paid-run consent rule. The preview is not approval.

`senders.mailboxes.deliverability_auto_enroll` is standing permission for new eligible mailboxes. Arming is approval-gated and bounded by `per_cycle_credit_cap`. It does not enroll mailboxes that already exist. Read the current arming state before changing it.

Disconnecting monitoring, when the schema offers it, is a separate preview. Do not cancel the mailbox to stop monitoring.

## What never follows from a test

- Do not unpause, raise a cap, or retire a domain from one seed report.
- Do not call placement on a paused mailbox and treat `refused_paused_mailbox` as a result about folders.
- Weigh bounce auto-pause and authenticated provider telemetry above a single seed test.
- A healthy-looking probe does not clear a bounce pause. Resume only after the cause is fixed and the list is clean, with the mailbox status capability the schema names, under the paid-run consent rule.
