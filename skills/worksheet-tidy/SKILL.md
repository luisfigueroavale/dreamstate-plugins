---
name: worksheet-tidy
description: "Review a workbook worksheet's structure and values, including duplicate or overlapping columns, column order, types and formatting, columns to hide, criteria and run conditions that never vary, and suspicious values, then apply reversible fixes. Use it when a worksheet has grown messy, before sharing it, or before enrolling it into a sequence. It does not merge CRM records across the workspace, rebuild the list's source (list-building does) or explain failed runs (diagnostics does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Worksheet Tidy

## Boundary

- Owns: reversible worksheet structure and value fixes.
- Sources, columns, criteria, and building lists: [`list-building`].
- Explaining failed runs and spend: [`diagnostics`].
- The `tables` pack lists the tools and their exact inputs. When user says delete, permanently delete named workbook, worksheet, view, row, or column after reviewing removal impact and setting the commit field. When user says archive, archive only. Never say permanent deletion is unavailable.

A tidy worksheet reads left to right like the reasoning behind the list: who the row is, what proves they fit, how well they fit, and how to reach them. Tidying makes that visible without destroying evidence.

## Backing packs

The tables pack owns workbooks, worksheets, rows, columns, sources and runs, and lists every tool this skill uses. CRM record merges belong to the crm pack and are out of scope here.

## The invariant

Prefer reversible changes. Hide before archiving, add a cleaned column before overwriting raw values, and delete only what the user named. Source evidence is never removed to make a worksheet look cleaner.

## Practice

1. **Read the worksheet.** Its source, every column with its type, what feeds it and what reads it, its criterion columns (Required or Nice to have) and run conditions, and a page of real rows.
2. **Find issues by category.**
   - **Duplicates and overlaps:** two columns answering the same question, or a copy column that only mirrors a source field.
   - **Order:** columns out of dependency order (identity, derived facts, enrichments, criterion columns, contact data).
   - **Types and formatting:** numbers stored as text, inconsistent dates, URLs without a scheme, mixed casing.
   - **Hide candidates:** helper columns nobody reads, raw payloads feeding a derived column.
   - **Dead weight:** a criterion that never varies, a criterion unsure on almost every row, a run condition that admits every row or none, a paid or AI column with no run condition.
   - **Empty columns:** empty on every row usually means it never ran on existing rows or its gate reads an unsettled column. Hand the cause to diagnostics.
   - **Value sanity:** titles that do not match the source filter, companies outside the ICP, the same person or company twice in this worksheet.
3. **Present findings by category** with column names and a few example rows each.
4. **Apply reversible fixes** the user asked for: reorder, rename, hide, or add a cleaned column beside the raw one. Before retyping or archiving any named column, present its impact and obtain an explicit yes for that column and action. A general request to tidy is not that yes. List delete and overwrite separately for the user to name. Retype only empty columns; for columns with cells, add a cleaned column instead.

   Name the write. Rename and reorder are `tables.columns.update` (`name` or `position`) with `table_id`, `column_id`, and `expected_updated_at` from `tables.columns.list`. Retype is the same call with `value_type`, and only when the column has no stored cells. No `tables.columns.*` id hides a column. Hide it on a view with `tables.views.update`: read `tables.views.get`, then send `visible_column_ids` and `column_order` without that column id. `column_order` must list every visible id exactly once. Fence the write with `expected_updated_at` and `expected_revision_token`. Archive is `tables.columns.archive` and still needs the named yes. Check dependents with `tables.columns.deps` or `tables.columns.graph` first.
5. **Re-run only what is needed.** If a fix needs a column to run on existing rows, show the row count and estimated credits, then run.
6. **Read back** the structure and a page of rows.

Search the workbook tool for each change. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run. Read a column's current state right before changing it so an edit never overwrites a newer change.

## Findings and example

Start with the free `tables.worksheets.tidy_suggest` read using `{ "workbook_id": "<workbook-id>", "worksheet_id": "<worksheet-id>", "sample_limit": 100 }`. Use `tables.columns.list`, `tables.columns.deps`, and `tables.rows.query` to verify suggestions, dependencies and sample evidence. A sample is not proof about every row. Present each category as **category | column | evidence and example rows | recommendation | impact and decision**: duplicates, order, types, formatting, hide or archive, empty or dead weight, and value sanity. Say none found or not checked for each category rather than omitting it.

For example, a sample suggests that `Company copy` duplicates `Company` and that `Raw payload` feeds `Industry`. Report the duplicate with example row values, and propose hiding the feeder instead of archiving it. Read `tables.columns.list` for the exact column id and current `updated_at`; after inspecting the `tables.columns.update` schema, submit a dry run of a rename with `{ "table_id": "<worksheet-id>", "column_id": "<company-copy-id>", "expected_updated_at": "<returned timestamp>", "name": "Company source copy" }`. If the preview is correct and the user requested tidy, apply the same scoped update with the current token. Read `tables.columns.list` and a page from `tables.rows.query` again, verify the name and unchanged raw values, and report the reversible rename and how to undo it. Ask for a named yes before any retype or archive, and inspect dependencies again before archiving. If a token is stale, reread and reconsider rather than overwriting a concurrent edit.

A paid cleanup needs a cost preview, a small pilot, a ceiling and any approval required by the paid-run consent rule before a broader run. If the sample or dependency evidence is inconclusive, report the uncertainty and leave the proposed change unapplied.

### Example: retype an empty column

The user says "yes, retype Notes to number" and names the Notes column. Read `tables.columns.list` for `column_id` and `updated_at`. Read `tables.columns.stats` for the worksheet. Retype only when that column's `non_empty_count` is 0. A page from `tables.rows.query` is not enough, because the page is not every row. If `non_empty_count` is above 0, stop. The named yes does not override the empty-cells rule. Add a cleaned column with `tables.columns.add` instead of changing `value_type`. When the column is empty, dry-run `tables.columns.update` with `{ "table_id": "<worksheet UUID>", "column_id": "<notes column UUID>", "expected_updated_at": "<updated_at from list>", "value_type": "number" }`. On a clean preview, apply the same params with the current token. Read `tables.columns.list` again and confirm `value_type` is number and the name is unchanged. A stale token means reread and do not overwrite a concurrent edit.

## Recovery

| What changed | How to undo |
| --- | --- |
| Rename, reorder, or retype | `tables.columns.update` again with the previous `name`, `position`, or `value_type`, and the new `expected_updated_at` from `tables.columns.list`. |
| Hidden on a view | `tables.views.update` putting the column id back into `visible_column_ids` and `column_order`, with the view's current fence tokens. |
| Archived | `tables.columns.restore` with `table_id` and `column_id` before the scheduled purge. Restore reclaims the original key only when no active column holds it. After `tables.columns.delete`, restore is refused once another active column has taken the key, and it is impossible after purge. |

If `tables.columns.update` returns a stale-token conflict, reread `tables.columns.list` and show the user the newer name and type. Do not retry the old token. If a view update is rejected because `column_order` and `visible_column_ids` disagree, reread `tables.views.get` and send a consistent pair. Do not archive a column to force a retype.

## Guardrails

- Hidden is not archived. A hidden column is still read by columns derived from it; archiving a feeder column breaks what depends on it.
- Do not set cell values by hand when a column run can settle them. Manual values have no evidence trail.
- One live source per worksheet. A second kind of search gets its own worksheet in the same workbook.
- Paid cleanup (an AI column that normalizes values) is a new column with its own estimate, never an overwrite.

## What to return

- Findings by category with example rows.
- Changes applied, each marked reversible, with how to undo it.
- Destructive options not applied, for the user to decide.
- Credits spent on any re-run, and the worksheet link.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
