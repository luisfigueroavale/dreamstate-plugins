---
name: diagnostics
description: "Answer one operational question with evidence, such as where a cell value came from or why it is empty, which runs failed and which are safe to retry, why a sequence has not sent, or where credits went and how to spend less. Use it when something in a worksheet, sequence or workflow looks wrong, stuck, missing or expensive. It reads before it changes anything. It does not redesign lists, sequences or workflows, or tidy worksheet structure (worksheet-tidy does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Diagnostics

## Boundary

- Owns: explaining cell values, failed runs, and unsent work, including why a sequence or a lead has not sent.
- Building or editing a sequence: [`outreach`].
- Sources and columns: [`list-building`]. Reversible worksheet fixes: [`worksheet-tidy`].
- Building and running automations: [`workflows`]. Reporting a product problem: [`feedback`].
- The `workspace` pack lists the tools and their exact inputs.

Good diagnosis answers one question with the least evidence that settles it. Pick the question, follow the object to its run, and separate "still working" from "broken".

## Backing packs

Generic run controls and run records use `runs_get`, `runs_events`, and `runs_cancel`, without a pack. The workspace pack explains credits, limits and action costs. The object itself belongs to its own pack: tables for cells and columns, sequence for sends, workflow for graph runs, unibox for conversations.

For questions about where credits went, call `workspace.usage.spend_breakdown` with the requested time window. Report unattributed spend and distinguish the overall total from the returned top groups.

CRM connection health, import rules, field mappings, and object-pair sync repair belong to [`crm`].

## The invariant

Read first, change second. Pending, queued, paced, capped and unknown are not failures, and none is zero. Never retry an external send (DM, email, post) whose outcome is unknown before checking whether it went out.

## Choose one question

| Question | Start from |
|---|---|
| Where did this value come from, or why is this cell empty? | `tables.cells.inspect` for one cell, then `tables.cells.history` if the current value is not the one you expected |
| Which rows or runs failed, and which are safe to retry? | Run history, then one run's events |
| Why has this sequence not sent? | Sequence state, enrollment, sender, sending hours, caps |
| Why did this worksheet cost so much, and how do I spend less? | The worksheet credit summary, then individual charged runs |

## Evidence order

1. **Open the object the user sees** and note its link.
2. **Read the owning run:** state, attempt, provider, credits charged, structured error.
3. **Classify:** running, queued, paced by a cap or sending hours, waiting on an upstream column, missing a connection, empty provider answer, failed with an error, or a likely product defect.
4. **Explain in plain words,** quoting the state or error that proves it.
5. **Propose the bounded fix:** retry only failed rows or the failed node, continue remaining rows instead of re-running a column, connect the named account, or move a gate. Say whether it spends credits or repeats anything external.
6. **Act and read back.** A retry that only spends credits runs after showing scope and estimate. Follow it to a final result.

Search the tool for the read you need. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run.

## Cell provenance

The named read for one cell is `tables.cells.inspect`. Params are `table_id`, `row_id`, and `column_id`. Optional `run_id` or `history_id` scopes the diagnostics. Pass at most one of them. It does not rerun the cell. The result is `cell` (nullable), `run_diagnostics` (up to 200), and `execution`. `execution.state` is `not_run`, `queued`, `running`, `completed`, `failed`, or `skipped`, plus `run_id`, `run_status`, `attempts`, `completed_at`, `skip_reason`, `failure_message`, and `billing`. In `billing`, `credits_saved` and `cache_hit` are not reported. Treat them as absent, not as zero or false.

Why a cell is empty:

- `cell` null or `execution.state` `not_run`: the column never ran on this row, often because it was added after the row landed.
- `skipped` with a `skip_reason`: a gate or run condition held it. Name the reason. Do not rerun until the gate input has settled.
- `completed` with an empty value: the provider returned nothing. That is not a failure. Do not repeat the same inputs.
- `failed`: quote `failure_message` and the `run_id`. Retry only when the failure class in the table below says a retry is safe.
- `queued` or `running`: still working. Do not retry.

`tables.cells.history` lists each stored write to that same cell, oldest first. Each event has the value before, the value after, when, who, the run id, and the provider record. `old_value` null means that write was the first one stored. Pass `next_cursor` back as `cursor`. This is the prior-value read. It is not the current value.

For many cells, `tables.rows.query` joins `cell_provenance` onto each row, including `manual_override`. A true `manual_override` is a static value that wins over the formula or source. `tables.cells.read_page` carries no diagnostics, so it cannot explain an empty cell. Do not call `tables.cells.inspect` once per row of a page.

For worksheet costs, discover and read the worksheet credit summary before drilling into
runs. Its decimal total and per-column breakdown cover the entire worksheet,
including historical executions. Check completeness and pending attempts: a
partial recorded total does not prove the remaining work was free. Current cell
values and the visible grid window do not represent cumulative spend.

## Common findings

- **Column empty on every row:** its gate reads a column that has not settled, or it was added after rows landed and never ran on them.
- **Criterion stuck unsure:** it waits on an enrichment column that failed or is still running.
- **Credits higher than expected:** a column re-run from the top billed the sample twice, contact data ran before the Required criteria, a paid or AI column has no run condition and spent on rows an earlier column already failed, or an expensive column sits before a cheap selective one.
- **Cell blocked by a daily limit:** workspace pacing, not a defect.
- **Sequence enrolled but silent:** not started, sender not connected, outside sending hours, or caps reached.

## Classify failures and refusals

| Evidence | Next action |
|---|---|
| Transient error or 5xx | Retry only failed items after checking scope and cost. Stop after the second identical failure. |
| Completed item with an empty value | Treat it as an empty provider result, not a failure. Do not repeat the same inputs expecting a different result. |
| Missing input across failed rows | Repair the input before retrying the affected rows. |
| `upgrade_required` | Explain the plan requirement and offer the upgrade path; credits alone do not resolve it. |
| `insufficient_credits` | Check the available balance and quote; reduce scope or request a top-up. |
| `rate_limit_exceeded` or rate limited | Wait for the returned retry time, then retry once. |
| `approval_required` or `spend_cap_required` | Show the proposed scope, cost and ceiling and obtain the required decision. |
| `budget_policy_exceeded` | Explain the applicable budget and request a permitted scope or policy decision. |
| Scope denied | Identify the missing scope or connection and obtain authorized access; do not bypass it. |
| Effect unknown | Reconcile each destination and receipt. Only replay after a human verifies every destination and the replay is safe; never replay in an unattended loop. |

Before paid work, read `workspace.billing.overview` with `{}` and the applicable quote or dry-run preview. Compare the balance with the estimated scope and maximum, including retries, which can reserve and spend credits again. If the quote is unavailable, report uncertainty and use a small bounded pilot only under the paid-run consent rule. Review the pilot before expanding, and preserve the approved ceiling. A reservation is not a charge: distinguish reserved, captured and released amounts when evidence provides them.

For an unknown social publish, read `social.content.artifact_get` and its deliveries, the original run and provider destination. A publishing or reconciling state is not proof of failure. Confirm the post exists or is absent at every destination before any replay; preserve the original operation identity and do not bypass a reconciliation lock. If absence cannot be proven, report the unresolved state.

## Worked example

A worksheet enrichment appears stuck. Read `workspace.billing.overview` with `{}` and preview `tables.table_runs.preview_cost` with `{"table_id":"33333333-3333-4333-8333-333333333333","binding_ids":["88888888-8888-4888-8888-888888888888"],"selection":{"kind":"view","view_id":"44444444-4444-4444-8444-444444444444","expected_view_updated_at":"2026-08-13T12:00:00.000Z"},"caps":{"max_credits":500}}`. Compare the returned estimate and balance, then inspect the run and failed items. If an item completed empty, report the provider miss; if it failed transiently, seek authorization for a bounded retry of failed items. Read the resulting run and cell before reporting the outcome. The example ceiling is illustrative, not an approval.

## Guardrails

- Stop reading when the question is answered; do not dump every log.
- A succeeded step with a receipt never runs again. Stop after the second identical failure and report the evidence.
- Use returned links rather than fabricating them. File feedback only with explicit consent.
- Do not hide a defect behind another provider or a manual cell edit. Name it and give the run and object identifiers to report.
- Cancel only a run the user wants stopped, then read its final state.

## What to return

- The question, a one or two sentence answer, and the evidence (link, run, state, quoted error).
- The fix, its cost, whether it repeats anything external, and its result if applied.
- Any product defect with the identifiers needed to report it.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.

## Sequence evidence

Read audience matches, eligible, duplicates, blocked rows, staged candidates, admitted people and actual sends as different measures. An unavailable count is not zero. An incomplete row sample or analytics read cannot establish a full-population total. A failed analytics request needs an explicit error and retry; cached totals need a freshness marker.

Separate invitation requests from DMs waiting for acceptance. A waiting-connection timestamp is the acceptance deadline, not an earliest send time: acceptance can wake the DM sooner. Distinguish a schedule window, a daily or rolling-week limit, reservations owned by existing obligations, a provider restriction, missing identity, a disconnected sender and a service failure. Future invitation commitments can reserve weekly room even if they use no daily room today. Preserve configured limits.

For a person, read the current step, pinned source and version, pause/stop reason, next check or deadline, attempt and receipt evidence before suggesting a repair. Unknown delivery requires reconciliation; proven absence is a different outcome. Resume only the system pause that the evidence allows to recover. Never resume manual stops, replies, unsubscribes or other recipient stops. Archived personalization sources and OOO without a reliable structured return date require visible review. Do not infer return dates from keywords or elapsed time.

## Reading sequence state

A lead with status running and a wait code is waiting, not paused. Read `wait.is_paused`, `wait.explanation` and `wait.resumes_when` on `sequence.sequences.contacts_list` and `person_timeline_get`; they say in words what the lead waits for and what ends it.

Read `sending_state` on `workflow.automations.get` for the sequence itself, and `sendability` on `sequence.sequences.sender_accounts_list` for each sender account. Say which one blocks sending before you suggest a repair.

A sequence with `status: published` is live. A `next_run_at` far in the future on a waiting_connection lead is the acceptance deadline, not a send time.

For a source run, `source_stop.explanation` says why it stopped. `source_stop.can_continue` says whether more rows can be loaded, and `source_stop.continue_with` names the call that loads them.
