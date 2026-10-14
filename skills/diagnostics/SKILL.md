---
name: diagnostics
description: "Answer one operational question with evidence, such as where a cell value came from or why it is empty, which runs failed and which are safe to retry, why a sequence has not sent, or where credits went and how to spend less. Use it when something in a worksheet, sequence or workflow looks wrong, stuck, missing or expensive. It reads before it changes anything. It does not redesign lists, sequences or workflows, or tidy worksheet structure (worksheet-tidy does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Diagnostics

## Boundary

- Owns: explaining cell values, failed runs, and unsent work.
- Sequence has not sent: [`outreach`].
- Sources and columns: [`list-building`].
- Reversible worksheet fixes: [`worksheet-tidy`].
- Building and running automations: [`workflows`].
The `workspace` pack lists the tools and their exact inputs.

Good diagnosis answers one question with the least evidence that settles it. Pick the question, follow the object to its run, and separate "still working" from "broken".

## Backing packs

Generic run controls and run records use `runs_get`, `runs_events`, and `runs_cancel`, without a pack. The workspace pack explains credits, limits and action costs. The object itself belongs to its own pack: tables for cells and columns, sequence for sends, workflow for graph runs, unibox for conversations.

## The invariant

Read first, change second. Pending, queued, paced, capped and unknown are not failures, and none is zero. Never retry an external send (DM, email, post) whose outcome is unknown before checking whether it went out.

## Choose one question

| Question | Start from |
|---|---|
| Where did this value come from, or why is this cell empty? | The cell, its column, the run that settled it |
| Which rows or runs failed, and which are safe to retry? | Run history, then one run's events |
| Why has this sequence not sent? | Sequence state, enrollment, sender, sending hours, caps |
| Why did this cost so much, and how do I spend less? | Finished runs and charged credits |

## Evidence order

1. **Open the object the user sees** and note its link.
2. **Read the owning run:** state, attempt, provider, credits charged, structured error.
3. **Classify:** running, queued, paced by a cap or sending hours, waiting on an upstream column, missing a connection, empty provider answer, failed with an error, or a likely product defect.
4. **Explain in plain words,** quoting the state or error that proves it.
5. **Propose the bounded fix:** retry only failed rows or the failed node, continue remaining rows instead of re-running a column, connect the named account, or move a gate. Say whether it spends credits or repeats anything external.
6. **Act and read back.** A retry that only spends credits runs after showing scope and estimate. Follow it to a final result.

Search the tool for the read you need. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run.

## Common findings

- **Column empty on every row:** its gate reads a column that has not settled, or it was added after rows landed and never ran on them.
- **Criterion stuck unsure:** it waits on an enrichment column that failed or is still running.
- **Credits higher than expected:** a column re-run from the top billed the sample twice, contact data ran before the Required criteria, a paid or AI column has no run condition and spent on rows an earlier column already failed, or an expensive column sits before a cheap selective one.
- **Cell blocked by a daily limit:** workspace pacing, not a defect.
- **Sequence enrolled but silent:** not started, sender not connected, outside sending hours, or caps reached.

## Guardrails

- Stop reading when the question is answered; do not dump every log.
- A succeeded step with a receipt never runs again.
- Do not hide a defect behind another provider or a manual cell edit. Name it and give the run and object identifiers to report.
- Cancel only a run the user wants stopped, then read its final state.

## What to return

- The question, a one or two sentence answer, and the evidence (link, run, state, quoted error).
- The fix, its cost, whether it repeats anything external, and its result if applied.
- Any product defect with the identifiers needed to report it.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
