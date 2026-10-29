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
4. **Apply reversible fixes** the user asked for: reorder, rename, hide, retype, or add a cleaned column beside the raw one. When the user says "tidy it", apply the reversible fixes and list destructive ones (archive, delete, overwrite) separately for the user to name.
5. **Re-run only what is needed.** If a fix needs a column to run on existing rows, show the row count and estimated credits, then run.
6. **Read back** the structure and a page of rows.

Search the workbook tool for each change. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run. Read a column's current state right before changing it so an edit never overwrites a newer change.

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
