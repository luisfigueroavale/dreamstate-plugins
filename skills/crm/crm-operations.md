# CRM lists, samples, bind, import, and sync

A CRM list is a worksheet. Lists use worksheet capabilities, not a separate list object.

Create the workbook with `tables.workbooks.create` when none exists, then the list with `tables.worksheets.create`. Rename with `tables.worksheets.update`. Archive with `tables.worksheets.archive`. Delete with `tables.worksheets.delete`. Deleting a list never deletes the records. Add one record with `crm.records.list_add` and `{ "record_id": "<record UUID>", "table_id": "<worksheet UUID>" }`. `changed: false` means the record was already a row. Remove with `crm.records.list_remove`. Read membership with `crm.records.lists_get` or `crm.records.memberships_get`.

## Sample records

Read `crm.records.get` with `{ "record_id": "<record UUID>" }` and inspect current values. The record is a sample when `_sample`, `sample`, or `is_sample` is true, 1, or the text true, t, or 1. Also exclude a person whose only email is on a reserved example domain (`example.com`, `example.net`, `example.org`, or a host under `example`). Leave sample records out of customer totals, pipeline counts, and sync audiences.

`crm.records.keep` with `{ "record_id": "<record UUID>" }` clears that sample flag on one record. Dry run returns `sample`, `kept`, and `already_real` and writes nothing. Live keep requires an idempotency key. Linked records stay flagged. Keep does not clear `is_demo`, `is_test`, or `sample_data`. Call keep only when the user asked to treat that one record as real.

## Bind, then promote

Scrub malformed emails before any bind or import. Read the email column with `tables.rows.query`. Hold out an address with no `@`, with whitespace inside the address, or on a reserved example domain. Report how many rows were held out. There is no separate email-scrub capability.

Bind-and-create is `crm.records.unbound_rows_enroll`. It creates person records only. It never creates companies and never writes worksheet cells. Dry run writes nothing and returns `would_bind` (one existing person, no field on that record changes), `would_create` (no match, a person will be created), skipped ambiguous rows (more than one match, left alone), and `no_identity`. `table_id` null means every eligible worksheet, so pass a specific worksheet. `limit` is 1 to 1000. `remaining` counts unprocessed rows that still have a usable identity.

```json
{ "table_id": "33333333-3333-4333-8333-333333333333", "limit": 25, "email_column_id": "12121212-1212-4212-8212-121212121212" }
```

Live enrollment is the truth write. Read the returned `enrollments` (`table_id`, `row_id`, `record_id`, `record_created`). If `remaining` is above zero, run another bounded batch. Do not continue while conflicts are unexplained.

`tables.columns.bind` is the other bind. It links rows you pass to existing records and never creates a record and never copies fields. Outcomes are `bound`, `ambiguous`, or `unmatched`. Use it when creates are not wanted.

Promote only after a bind column already resolved the row. `crm.records.promote` has no dry run. Only live mode writes. It never resolves a row and never writes bind links.

```json
{
  "table_id": "33333333-3333-4333-8333-333333333333",
  "bind_column_id": "dddddddd-dddd-4ddd-8ddd-dddddddddddd",
  "object": "person",
  "map": [{ "target_attr_key": "title", "source_column_id": "eeeeeeee-eeee-4eee-8eee-eeeeeeeeeeee" }],
  "policy": "fill-empty-only",
  "rows": [{ "row_id": "11111111-1111-4111-8111-111111111111", "values": { "eeeeeeee-eeee-4eee-8eee-eeeeeeeeeeee": "VP Engineering" } }]
}
```

`fill-empty-only` is the default policy. `overwrite` and `skip-conflicts` need an explicit ask. `skip-conflicts` writes empty fields, treats an identical value as a no-op, and refuses a differing populated value as `skip_conflict`. Never map an identity field onto a value that would replace a human-entered email or profile URL. Read back with `crm.records.get` on one returned `record_id` and confirm `title` filled a blank and the email was unchanged.

Ongoing flow from later cell edits is a different call: `tables.columns.crm_target_attach` with `write_policy` `fill_blank_only`. That queues completed cells. A missing queue outcome is not proof the record changed. Read the record.

## Import

`crm.record_imports.source_upload` stores the CSV and returns a mapping preview from the first 20 data rows. It does not create a job and does not stage rows. Params are `object_definition_id`, `file_name`, `mime_type`, and `base64`. Read `headers`, `sample_rows`, `mapping_preview`, and `source_checksum`. If a sample email is malformed, fix the file or omit those rows. Do not stage them.

`crm.record_imports.create` binds the checksum, mapping, and duplicate policy. It stages nothing. Pass `queue` false until the staged rows are the scrubbed set. `operation_mode` `create` with `duplicate_mode` `skip` leaves existing matches untouched. Then `crm.record_imports.rows_stage` with `job_id` and only the cleaned rows (`source_row_number`, `data`). Read back with `crm.record_imports.get`: `status`, `processed_rows`, `succeeded_rows`, `failed_rows`, `skipped_rows`. Failures are `crm.record_imports.errors_list`. A multi-match row is skipped and logged, never guessed.

## Sync ladder

`crm.sync.configure` previews one local and remote object pair. Objects are `company`, `person`, or `deal`. Defaults to dry run, which writes nothing and returns `saved` false plus `next_cycle` (`inbound_reads`, `outbound_writes`, `bounded`, `read_limit`). Inbound and bidirectional previews call the provider with a bounded read.

```json
{ "local_object_key": "person", "remote_object_key": "person", "direction": "inbound", "dry_run": true }
```

`crm.sync.enable` and `crm.sync.disable` are the standing HubSpot or Attio pair. Provider is `hubspot` or `attio`. HubSpot objects are `contacts` or `companies`. Attio objects are `people` or `companies`. Salesforce is not a provider on these two calls. Use the integrations connection path for Salesforce.

1. Read `crm.sync.status` with `{ "provider": "hubspot", "object": "contacts" }`, or with `run_id` alone. Do not pass both selectors. This read is free.
2. Preview with `crm.sync.configure` dry run, and with `crm.sync.enable` in dry run. Dry run enable performs no provider call.
3. A live enable follows the paid-run consent rule: the schema's row ceiling and credit ceiling, an idempotency key, and approval when the rule requires it. Read the exact field names with tools_get. `no_run_now` true arms the cron without queueing the immediate first cycle. The queued first cycle does not itself write to HubSpot or Attio.
4. Read back with `crm.sync.status` and report cadence, watermarks, and last-cycle counts. A missing count is not zero.

`crm.sync.disable` with `{ "provider": "hubspot", "object": "contacts" }` stops the cron and revokes the standing write. Direction, cadence, mappings, and watermarks stay. Dry run previews the stop. It does not delete the config and does not call the provider.
