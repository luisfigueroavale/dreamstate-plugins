# Reading the worksheet back

A write is a request. The read back is the evidence. Nothing is reported as landed, enriched, judged or scored until it has been read off the worksheet after the run finished.

Read rows, not counts. One page read of the rows with the columns you need is the read back; opening a single cell's detail is for diagnosing that cell (its run, provenance or error), never a way to read rows one at a time.

A count that is still being computed is unknown, not zero, and a completion claim made on it has no evidence behind it. When the number matters, ask for the exact count.

Every cell says where its value came from and what state it is in. Empty and failed are different answers and are reported separately; neither is zero. A missing provider answer is unknown, and so is a Required column with no data. Neither is false, and neither may be filled in from a similar row.

Let columns settle cells. Setting cell values by hand bypasses the evidence trail and is worth avoiding when a column run can do it.

## After a failed or partial run

Read `tables.table_runs.get` for status, credits, `state_version`, and `checkpoint`. Read `tables.table_runs.failure_report` for each row's outcome, `error_category`, `retryable`, and `why`. Read `tables.table_runs.effect_unknown` before any continuation. Then `tables.cells.read_page` for the rows you will report. A page you requested and did not read is not evidence.

Succeeded, failed, skipped, and empty stay four counts. Do not add them into one total and call the run complete. An empty cell is unknown. It is not false, and it is not a copy of a neighbour row.

When `checkpoint.resumable` is true, the continuation is `tables.table_runs.resume`. That call spends again only for the remaining work. `tables.table_runs.retry` is the other door: a fresh run of the same binding, a new spend, and nothing reused. The retry receipt's `run_id` is the original run id, so the next read is `tables.table_runs.get` on that id. Neither door is safe when `tables.table_runs.effect_unknown` lists the run.

## What a completion report needs

The source run finished, landed and duplicate counts known, every enrichment run finished, then a worksheet page actually read. A queued source, a missing duplicate count, an unfinished enrichment or a page requested but never looked at is incomplete evidence.

Cite the workbook id, the worksheet id, the source run id, the landed count and the duplicate count from the terminal source receipt, each column run id, the qualified count, the credits on those runs, and the page of rows whose columns the decision uses. Quote a match count only when `tables.sourcing.preview` returned it, and name that preview.
