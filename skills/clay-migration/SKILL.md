---
name: clay-migration
description: "Move work from Clay into Dreamstate: read what exists, import tables and rows, rebuild columns as native worksheet columns and waterfalls, and verify coverage row by row."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Clay migration

## Boundary
- Owns: CSV-based Clay column classification, reconstruction and comparison.
- Waterfall enrichment columns on the rebuilt worksheet: [`enrichment`].
- The `tables` pack lists the tools and their exact inputs.

Use this for a user-supplied Clay CSV export, with or without a separate Clay column-settings JSON. Rebuilding people or company lists from search belongs to [`list-building`]. Running waterfall providers after columns exist belongs to [`enrichment`]. Durable schedules belong to [`workflows`].

The catalog has `tables.clay_migrations.preview`, `tables.clay_migrations.apply`, and `tables.clay_migrations.compare`. It has no Clay connect, discover, transfer, or hosted-workspace inventory capability. A CSV is not a workspace migration. Say that at the start. Cell values are evidence. They are not formulas, prompts, provider order, or HTTP configuration.

## Route

| Have | Do |
| --- | --- |
| CSV only | Preview, classify, import raw columns, apply only entries that have real configuration, compare after a pilot |
| CSV plus `clay_table_json` | Preview with both. Trust metadata over header-name guesses. |
| Ask to pull the Clay account | Stop. No capability does that. Ask for the export file. |
| Paid enrichment on rebuilt columns | [`enrichment`], after apply, with the paid-run consent rule |

Full CSV chain, classification, and the verification report: [csv-path.md](csv-path.md), [classification.md](classification.md), [verification.md](verification.md).

## Safety ladder

1. `tables.clay_migrations.preview` is free and read-only. Pass exactly one of `csv_content`, `upload_id`, or `attachment_id`. Add `clay_table_json` only when the user supplied formula, prompt, provider, or HTTP configuration.
2. Import raw rows with the CSV chain. Landing rows writes the worksheet and spends no provider credits. `replace_existing_rows` true soft-deletes every live row first. Do not set it on a worksheet that already has other rows. A missing confirm token returns 409 `csv_replace_confirmation_required`. That refusal is the guard.
3. `tables.clay_migrations.apply` creates columns in dependency order and does not run providers. Columns are created disabled. It does not overwrite a column key that already exists. It returns `execution_disabled` true and `cost_previews` pointing at `tables.table_runs.preview_cost`.
4. Price each created column with `tables.table_runs.preview_cost`. Follow the paid-run consent rule. Pilot a handful of rows through [`enrichment`] or the column run. Keep schedules and auto-run off.
5. `tables.clay_migrations.compare` reads up to ten existing rows. It does not spend.

## Recovery

| What happened | What to do |
| --- | --- |
| `csv_upload_not_found` or `csv_upload_invalid` | The upload has no headers or is the wrong workspace. Create the upload again. Do not apply. |
| `clay_migration_configuration_required` | No executable column in the plan. Read `remaining_manual_questions`. Preview again after configuration. Do not invent a prompt from cell values. |
| `clay_migration_dependency_cycle` | Fix dependencies and preview again. Do not apply a sliced plan that hides the cycle. |
| Formula question names an unsupported function | Leave the raw column. Rewrite only the functions the question lists. |
| HTTP question says non-GET is blocked pending explicit approval | Do not apply that column as GET. A non-GET needs its own approval after a real HTTP column exists. |
| Apply created zero new columns because keys exist | Read `tables.columns.list`. Do not delete destination columns to force a rewrite. |
| Compare fill rate is high | Fill rate is not semantic equivalence. Exact kinds report `match_rate`. |
| Uncertain provider run after a pilot | `tables.table_runs.effect_unknown` for that run. Do not retry blindly. |

Account for every known column as copied (raw value landed), rebuilt (disabled column created), needs decision (`manual_questions`), unsupported, or failed. Do not mark the export complete.

## Worked example

Preview this export. The formula is in `clay_table_json`, not in the cells.

`tables.clay_migrations.preview`:

```json
{"csv_content":"Full Name,Company Domain,Formula: Site\nAda Lovelace,example.com,example.com","clay_table_json":{"columns":[{"header":"Formula: Site","clay_type":"formula","formula":"lower({Company Domain})"}]}}
```

Read `suggested_row_kind`, `manual_questions`, and `column_plan`. Expect `full_name` and `company_domain` as manual raw keys, and `formula_site` as kind `formula` with `proposed_definition_id` `column.formula` when the expression validates. The rebuilt key is `formula_site_rebuilt`. If `manual_questions` is not empty, stop and answer them before apply.

Create the workbook if needed with `tables.workbooks.create`, then open the upload on the returned `table_id`. The chain and the concrete ids are in [csv-path.md](csv-path.md). After rows land, apply the plan preview returned, with no substituted placeholders:

```json
{"table_id":"4e7b2a10-6c5f-4d2b-9a71-0e3c8f6b2d14","column_plan":[{"source_header":"Full Name","raw_column_key":"full_name","kind":"manual","proposed_definition_id":null,"proposed_binding":null,"dependencies":[],"confidence":"manual","manual_questions":[],"rebuilt_column_key":"full_name_rebuilt"},{"source_header":"Company Domain","raw_column_key":"company_domain","kind":"manual","proposed_definition_id":null,"proposed_binding":null,"dependencies":[],"confidence":"manual","manual_questions":[],"rebuilt_column_key":"company_domain_rebuilt"},{"source_header":"Formula: Site","raw_column_key":"formula_site","kind":"formula","proposed_definition_id":"column.formula","proposed_binding":{"expression":"lower({{company_domain}})"},"dependencies":[],"confidence":"exact","manual_questions":[],"rebuilt_column_key":"formula_site_rebuilt"}]}
```

Use the worksheet id you actually created and the `column_plan` array preview actually returned. The object above is the shape for this sample when preview agrees. Read `created_columns`, `remaining_manual_questions`, `execution_disabled`, and `cost_previews`. Then `tables.table_runs.preview_cost` for `formula_site_rebuilt` before any run.

After an approved pilot of at most ten rows, compare:

```json
{"table_id":"4e7b2a10-6c5f-4d2b-9a71-0e3c8f6b2d14","column_plan":[{"source_header":"Formula: Site","raw_column_key":"formula_site","kind":"formula","proposed_definition_id":"column.formula","proposed_binding":{"expression":"lower({{company_domain}})"},"dependencies":[],"confidence":"exact","manual_questions":[],"rebuilt_column_key":"formula_site_rebuilt"}],"row_limit":10}
```

Read `rows_compared` and each column's `metric`, `match_rate`, `fill_rate`, and `credits_per_row`. Formula and provider waterfall use `match_rate`. AI, research, and other free text use `fill_rate`. Null raw and null rebuilt never count as a match. Write the verification report in [verification.md](verification.md) before you call the migration done.

## Decisions

- Raw values stay. Rebuilt columns are additional and disabled.
- Approximate waterfall provider order is a substitution. Say so. Dreamstate does not replay Clay's provider list as executable order.
- Missing configuration stays a question. Header keywords are a classification hint, not a recovered prompt.
- Compare is a sample of at most ten rows, not a full parity proof.
