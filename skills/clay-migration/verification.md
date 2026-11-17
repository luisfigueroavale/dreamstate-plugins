# Verification report

Write this after apply and, when a pilot ran, after compare. Claim only the rows you counted.

```text
Clay CSV verification
Source file:            <file_name>
Source data rows:       <count in the file, excluding the header>
Upload id:              <upload_id>
Landed / skipped:       <from csv_upload_land_chunk>
Destination worksheet:  <table_id>
Suggested row kind:     <person or company>

Column                        Disposition        Evidence
Full Name                     copied             raw manual column full_name
Company Domain                copied             raw manual column company_domain
Formula: Site                 rebuilt            formula_site_rebuilt, confidence exact, execution_disabled true
<header>                      needs decision     quote manual_questions
<header>                      unsupported        no catalog equivalent, raw column kept
<header>                      failed             error code and which call

Apply readback: created_columns, remaining_manual_questions, execution_disabled, cost_previews
Pilot:          column, row count, preview_id, paid-run consent, run_id, status
Compare:        rows_compared, metric, match_rate or fill_rate, credits_per_row
Limits:         no hosted Clay inventory; compare is at most ten rows; fill rate is not equivalence
```

Dispositions:

| Disposition | When |
| --- | --- |
| copied | Raw header landed as a manual cell. Always true for a landed CSV column. |
| rebuilt | Apply returned the column in `created_columns` and `execution_disabled` is true. |
| needs decision | The plan entry has `manual_questions`, or apply returned them in `remaining_manual_questions`. |
| unsupported | Kind manual with no definition, or a function or method the preview refused. |
| failed | A call returned an error. Name `clay_migration_configuration_required`, `clay_migration_dependency_cycle`, `csv_upload_not_found`, `csv_upload_invalid`, or `csv_replace_confirmation_required`. |

`match_rate` is for `formula` and `provider_waterfall`. Both raw and rebuilt must be set and stable-equal. Empty cells do not match. `fill_rate` is the share of compared rows with a rebuilt value, used for AI, research, and other free text. `credits_per_row` comes from cell receipts on rebuilt keys. Pass `clay_credits_per_row` keyed by source header only when the user gave Clay's own cost. Otherwise leave that comparison null.

Do not delete raw columns after a high fill rate. Do not turn columns on, enroll a sequence, or enable a schedule inside this report. Those are separate approvals in [`enrichment`], [`outreach`], and [`workflows`].
