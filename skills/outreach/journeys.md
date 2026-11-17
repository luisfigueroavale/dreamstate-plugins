# Journeys

How a running sequence branches, when it is allowed to send, and how to change it without rewriting a send that already left.

Read `sequence.sequences.step_options` before adding a step. Then `workflow.automations.node_inspect` for that subtype. Do not invent a field the inspect result does not list. A reply, an unsubscribe, or a failed send already ends that lead, so do not add a condition whose only job is to test reply.

## Branches

A condition step sends nothing. It needs `kind` `condition`, `expression`, `true_step_id`, and `false_step_id`. Either target may be null, and null ends that arm. There is no separate stop step.

`expression` is one of these shapes:

- A field leaf `{ "field", "operator", "value" }`. `operator` is `eq`, `neq`, `exists`, `not_exists`, or `in`. `eq` and `neq` require `value`, or the write returns `condition_value_required`. `in` requires an array, or the write returns `condition_in_requires_array`. `exists` and `not_exists` ignore `value`. `field` is a field on the enrolled subject. After a completed call, `call_disposition` is one of those fields.
- A signal leaf `{ "signal": "<name>" }`. The name is read from that enrollment's `trigger_signal_json`. It decides routing. It does not permit naming a private signal in copy. Copy still follows the public signal rule in SKILL.md.
- A group `{ "all": [ ... ] }` or `{ "any": [ ... ] }` of the same predicates.

A `random_split` step also sends nothing. `branches` holds 2 to 4 objects, each `{ "weight", "next_step_id" }`. Each `weight` is an integer from 1 to 100, and the weights total exactly 100. The enrollment id plus the step id picks the branch, including on retry, so a dry run shows the branch that enrollment will keep.

On `workflow.automations.node_add`, `branch` `"false"` attaches the new step to a condition's false arm. Omit `branch` for the true arm or the single next step. Pass `kind` `sequence` and the `expected_version` from `workflow.automations.get`. A stale token returns `context_conflict`. Read the sequence again and retry with the version that read returned.

Keep step ids stable. Later steps and both condition arms point at them. `workflow.automations.node_patch` refuses a change to `id`.

## Send windows and timezones

The sequence `timezone` is an IANA name, for example `America/New_York`. It is how that sequence's dates and times are evaluated. It does not decide the hours a sender may send. Sender accounts own delivery windows. There is no sequence send-schedule capability. `senders.policy.update` rejects `send_window_start`, `send_window_end`, `send_weekdays_only`, and `work_days`.

A closed window is a hold, not a pause. Do not pause the sequence because one row is outside hours. Recovery for a throttled row is the window ladder in stop-and-suppression.md.

Email hours are on the mailbox. Read `senders.mailboxes.list`, then `senders.mailboxes.update` with `mailbox_id` and at least one setting. Supplying only `mailbox_id` returns `At least one mailbox setting must be supplied`. `work_days` is one to seven entries, each `{ "day", "enabled", "ranges" }`. `day` is `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`, or `sunday`. `ranges` is one to six `{ "from", "to" }` values in `HH:MM`, and `from` must be earlier than `to`. A bad clock returns `expected HH:MM`. A backwards range returns `range from must be earlier than to`. An empty `work_days` uses the mailbox default. The clock is `send_timezone` when set, otherwise the workspace timezone. A closed mailbox window holds as `send_window`. A throttled email row that `sequence.sequences.retry_deferred` can requeue carries `email_send_window_hold`.

LinkedIn hours are on the account. Read `senders.linkedin_accounts.settings_get`, then write `work_days` with `senders.linkedin_accounts.settings_update`. A closed account window holds as `outside_account_send_window`. That reason is what `sequence.sequences.retry_deferred` can requeue. When the lead has a timezone, those weekday ranges are also applied in the lead's zone. Outside that zone the hold is `recipient_send_window`. An unusable zone is `recipient_timezone_invalid` and fails closed. Do not guess a zone to clear it.

WhatsApp hours use `senders.whatsapp_accounts.update`. Instagram hours use `senders.instagram_accounts.update`. Both accept `work_days`, `timezone`, and `daily_reset_timezone` on that account. A closed window holds as `send_window`. Read X hours with `sequence.x_automation_settings_get` before assuming a window. Do not copy an email mailbox window onto X.

`date_timing` is not a send window. It schedules one step from a worksheet timestamp: `{ "value": { "kind": "worksheet_column", "column_key" or "column_id", "path" }, "relation": "before" | "at" | "after", "offset_seconds", "missed": "skip" | "run_immediately" }`. The cell must be an ISO timestamp. A date with no time is refused, because no timezone is assumed. `offset_seconds` is 0 when `relation` is `at`, and otherwise 0 through 31536000. When `date_timing` is set, `delay_seconds` must be 0 or omitted, or the write returns `delay_seconds must be 0 or omitted when date_timing is set.` `delay_seconds` is seconds on the step itself, from 0 through 31536000. A standalone wait step is not authorable.

## Changing a running sequence

Read `workflow.automations.get` first and keep `expected_version`.

What can change on a sequence that is published, started, not paused, and not archived: step copy, `delay_seconds`, `date_timing`, condition expressions, branch targets, and new steps, through `workflow.automations.node_patch` or `workflow.automations.node_add`. That write saves a draft and, when the sequence is running, publishes it so current enrollments move at the next step. The write body does not include that publish receipt. Read `workflow.automations.get` after it. If `published_version_id` equals the new `sequence_version_id`, the running sequence took the edit. If `latest_draft_version_id` is the new id and `published_version_id` is still the old one, the edit stayed a draft. That happens when the sequence is paused, never started, or archived, and also when the live publish was refused. Do not retry the patch in a loop. Tell the user the running version did not change.

What a pause does: `sequence.sequences.pause` stops delivery. It does not unlock extra fields, and it stops the automatic publish. The draft sits until `sequence.sequences.publish`. That call sends nothing. Its params are `sequence_id` and `sequence_version_id` (the draft id `workflow.automations.get` returns as `latest_draft_version_id`). It does not take a migrate flag. Current enrollments stay on the version they already have. New enrollments use the published draft. Pause before an edit only when sends must stop while the draft is still unreviewed. Do not pause merely to edit copy.

What does not re-render: a step that is claimed, a send attempt in `attempting` or `reconciling`, or a step execution in `prepared`, `submitted`, `provider_unknown`, or `confirmed`. Those enrollments keep the old version until that action finishes, stored as `pending_sequence_version_id`. When the cursor moves, they take the new version only if the next step id exists there. If it does not, the event reason is `republish_continuation_removed`.

What to do instead of deleting the current step: enrollments track position by step id. If the current step id is missing from the new version, that enrollment is `kept_step_removed` and stays on the old version. To drop a wait, set `delay_seconds` to 0 and clear `date_timing`. A pure delay hold then releases. A LinkedIn connection wait, a pause, `date_timing`, or a nonzero delay stays. `sequence_republish_version_superseded` means another publish landed first. Read the published version before editing again.

Copy renders at send from the version the enrollment is pinned to. `sequence.sequences.copy_preview` uses that same renderer and sends nothing. Preview the new body before the patch when the sequence is already running, because the patch publishes. Read the saved step back after the patch. A prepared or confirmed send keeps the body it already rendered. Editing the worksheet after enrollment does not rewrite that prepared body.

## Worked example

The day-3 email is step `email-2`. The sequence is running. Cut its wait and replace the body. People already inside that send keep the old body.

1. `workflow.automations.get` with `{ "kind": "sequence", "automation_id": "<sequence uuid>" }`. Keep `expected_version` and the step id.
2. `workflow.automations.node_inspect` for the email subtype before writing fields.
3. `sequence.sequences.copy_preview` with `{ "sequence_id": "<sequence uuid>", "record_ids": ["<worksheet row uuid>"] }`. Read `resolved` body. This sends nothing.
4. `workflow.automations.node_patch` with `{ "kind": "sequence", "automation_id": "<sequence uuid>", "expected_version": <n>, "node_id": "email-2", "fields": { "delay_seconds": 0, "body_template": "Hi {{first_name}}, the pilot is open this month.", "fallback_template": "Hi, the pilot is open this month." } }`.
5. Readback: `workflow.automations.node_patch` returns `sequence_version_id` and `expected_version`. Then `workflow.automations.get`. `published_version_id` equal to that new `sequence_version_id` means running enrollments move at the next step. A newer draft id with the old `published_version_id` means the running version did not change. `context_conflict` means re-read `expected_version` and patch once. `sequence.sequences.copy_preview` again shows the body a later unsent step will render. It does not rewrite a step already `prepared` or `confirmed`. Do not resend that step.
