# Preview, one row, readback, effect_unknown

Use this only when a worksheet column already calls the HTTP API. Attaching a credential does not create that column.

## Ladder

1. `tables.columns.list` on the worksheet. Confirm the column id and that it has a binding. `tables.columns.bind` does not attach an HTTP binding. It links rows to CRM records. Do not use it here.
2. `tables.table_runs.preview_cost` with that column and a one-row selection. Read `preview_id`, `preview_digest`, `estimated_rows`, `estimated_credits` (minimum and maximum), and `expires_at`. No run is created and no provider is called.
3. One live row through `tables.columns.run` or `tables.table_runs.create`, using the paid-run consent rule and the quote you just read. `tables.table_runs.get` is the readback: `status`, `progress_json`, `completed_units`, `total_units`, `credits`, `stop`, and `checkpoint`. Per-row failures are `tables.table_runs.failure_report`, not the get payload.
4. Widen the selection only after that row's stored cell matches the expected output.

Upstream API charges are the user's even when the workspace quote is 0. Say so before the live call.

## effect_unknown

Call `tables.table_runs.effect_unknown` with `table_id`, `run_id`, and optional `limit`. The result is receipts still in progress or unknown. Unresolved means not safe to retry.

| Evidence | Next step |
| --- | --- |
| Receipts returned | Stop. Ask for the provider reference. Then `tables.table_runs.reconcile_receipt` with preview first. |
| No receipts, `tables.table_runs.get` says failed, `failure_report` names a definite error | Fix the input. A new one-row run is a new attempt, not a retry of the unknown call. |
| `tables.table_runs.retry` is the temptation | Do not use it while any unknown receipt exists. Retry calls the provider again and spends again, and the returned `run_id` is the original run id. |

`tables.table_runs.provider_summary` is the credit readback: ceilings, receipt totals, and ledger totals. Use it before saying what the run cost.

Reconcile params that must match the schema: `receipt_id`, `reconciliation` (`outcome` `completed` or `not_completed`, `provider_reference`, `observed_at`), `page_canonical_json`, `page_digest`, and `actual_cost_credits` as a decimal string. Do not invent a provider reference. A cost above the stored reservation is rejected.
