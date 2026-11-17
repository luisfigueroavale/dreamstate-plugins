# CSV path

Preview can run before any worksheet exists by passing `csv_content`. Import needs a worksheet. Order:

1. `tables.clay_migrations.preview` with the file text or `attachment_id`, plus `clay_table_json` when the user supplied settings.
2. `tables.workbooks.create` or `tables.worksheets.create` if there is no destination. Record `table_id`.
3. `tables.table_sources.csv_upload_create` with `table_id`, `file_name`, `headers`, and `row_kind` from `suggested_row_kind`. For Clay, keep unmapped columns (the default). Headers are trimmed, non-empty, and at most 200. Duplicate headers stay, with deduped keys.
4. `tables.table_sources.csv_upload_stage` with `upload_id` and `rows` (`source_row_number` plus `data`), or `attachment_id` for a chat file. Live mode only. Dry run cannot see a new `upload_id`. Stage until every source row is accepted. Restaging the same `source_row_number` does not double count.
5. `tables.table_sources.csv_upload_map` with the `suggested_mapping` you accept, `identity_column`, and `keep_unmapped_columns` true. Every raw header still becomes a manual column.
6. `tables.table_sources.csv_upload_attach` with `table_id`, `upload_id`, and `source_label`. Leave `replace_existing_rows` false unless the user named that worksheet and accepted the soft delete. The first call without `confirm_token` returns 409 `csv_replace_confirmation_required`.
7. `tables.table_sources.csv_upload_land_chunk` with `table_id`, `upload_id`, and `limit` (default 250, maximum 400) until `done` is true. Read `landed`, `skipped`, and `remaining`. This is the step that makes rows visible. No provider call and no credits.
8. `tables.clay_migrations.apply` with `table_id` and the `column_plan` from preview.
9. Price, pilot, then `tables.clay_migrations.compare`.

`tables.table_sources.csv_upload_delete` removes a staging or mapped upload only. It cannot delete rows after land.

## Worked numbers for the sample export

File text:

```text
Full Name,Company Domain,Formula: Site
Ada Lovelace,example.com,example.com
```

`tables.table_sources.csv_upload_create`:

```json
{"table_id":"4e7b2a10-6c5f-4d2b-9a71-0e3c8f6b2d14","file_name":"clay-export.csv","headers":["Full Name","Company Domain","Formula: Site"],"row_kind":"company","keep_unmapped_columns":true}
```

Set `row_kind` from preview's `suggested_row_kind`. For this sample the person test does not match, because it looks for contact, first name, last name, person, or linkedin profile, and `Full Name` is none of those. Preview returns `company`. Pass that.

Read `upload_id` and `suggested_mapping`. Stage:

```json
{"upload_id":"a17c9e02-3f48-4b6d-8e12-9c05b7a4d63f","rows":[{"source_row_number":1,"data":{"Full Name":"Ada Lovelace","Company Domain":"example.com","Formula: Site":"example.com"}}]}
```

Use the `upload_id` create returned. Map using `suggested_mapping`, with `identity_column` set to a header that is also a mapping key. Attach:

```json
{"table_id":"4e7b2a10-6c5f-4d2b-9a71-0e3c8f6b2d14","upload_id":"a17c9e02-3f48-4b6d-8e12-9c05b7a4d63f","source_label":"Clay export"}
```

Land:

```json
{"table_id":"4e7b2a10-6c5f-4d2b-9a71-0e3c8f6b2d14","upload_id":"a17c9e02-3f48-4b6d-8e12-9c05b7a4d63f","limit":400}
```

Read `landed` 1, `remaining` 0, `done` true before apply. Source row count is 1. If `landed` plus `skipped` is not 1, stop. A short land is a partial import, not a worksheet to rebuild on.

Count three ways and keep them separate: rows in the Clay file, rows `landed`, and rows `tables.clay_migrations.compare` reports as `rows_compared`. Compare never exceeds ten rows even when the worksheet is larger.
