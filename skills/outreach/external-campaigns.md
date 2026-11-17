# External campaigns

No `sequence.*` capability pushes contacts into Instantly, Lemlist, Smartlead, or any other sender tool. `sequence.sequences.external_push_status` only reads webhook delivery for sequence step sent events. It does not push, retry, or return the signed body. `limit` is 1 to 200 and defaults to 50. `pushes` lists `delivery_id`, `enrollment_id`, `status`, `attempt_count`, `last_status_code`, and `last_error`. `push_coverage.email` is unavailable because email sends do not enqueue this event.

The push is `integrations.outreach_connector_job_run`. It is not a sequence. Cadence, reply stop, and sending stay on that platform. Dreamstate only hands reviewed contacts to a campaign that already exists there.

Read `integrations.outreach_connectors_list` first. `handoff` is legal for `instantly`, `lemlist`, and `smartlead` only when that provider's row is connected. `export` is HubSpot, Attio, Salesforce, and Clay. Do not use export to mean a campaign push. Do not guess a campaign id. The destination is the existing campaign or sequence id at that provider.

Params: `provider`, `mode` `handoff`, non-empty `destination`, and exactly one of `contact_ids` (1 to 100 uuids) or `selection_snapshot_id`. Both or neither returns `Exactly one non-empty connector audience source is required`. `field_mappings` is 1 to 200 entries when you override the mapping. Its `update_policy` accepts only `always`.

Consent: this leaves the workspace. Dry run first and show the destination, the row count, and the mapping. A live call follows the paid-run consent rule. `contact_ids` writes before the response returns, so `job_id` is a record of work already done, not a job to poll. `selection_snapshot_id` on a handoff returns the destination and mapping plan, and a live handoff is refused with `Worksheet external sequencer handoff is preview-only.` Use `contact_ids` for a reviewed pilot, or build a native sequence instead. An asynchronous export, which is not this handoff, is polled with `integrations.outreach_connector_job_get`.

Stop when the connector is missing, the destination id was not read from the provider, the copy is unsupported, or the live handoff is not approved. A failed row is retried by id. Do not push the successful rows again.

## Worked example

Push 20 already reviewed contact ids into an Instantly campaign whose id was returned by the connector, not typed from memory.

1. `integrations.outreach_connectors_list` with `{}`. Proceed only when `provider` is `instantly` and `connected` is true, and `capabilities` includes handoff.
2. Dry run `integrations.outreach_connector_job_run` with `{ "provider": "instantly", "mode": "handoff", "destination": "<campaign id from the connector>", "contact_ids": ["<uuid>", "..."] }`. Read the destination and the count.
3. Live run uses the same params under the paid-run consent rule. Readback is `job_id`, `status`, `processed`, `succeeded`, `failed`, and `failed_rows`. Those writes have already happened. Do not poll `job_id` as if the send were still queued. A refusal `Worksheet external sequencer handoff is preview-only.` means the audience was a `selection_snapshot_id`. Switch to `contact_ids` or stop. `sequence.sequences.external_push_status` will not show this Instantly handoff.
