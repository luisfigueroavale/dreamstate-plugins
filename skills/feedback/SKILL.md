---
name: feedback
description: "File one agent feedback report after a task, when a capability is missing, a tool is broken or confusing, or a workaround was required. Use it to submit, list, or delete those reports. It does not explain the failure (diagnostics does) and diagnosis never submits on its own."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Feedback

## Boundary

- Owns: one feedback report per task, filed after the work finishes or stops.
- Explaining the failure itself: [`diagnostics`].
- The `workspace` pack lists the tools and their exact inputs.

File a report when a capability is missing, a tool is broken or confusing, or you had to work around something. Call `workspace.feedback.submit` once per task, after finishing or giving up. Diagnosis never submits on its own.

## Backing packs

`dreamstate feedback submit`, `dreamstate feedback list`, and `dreamstate feedback delete` call `workspace.feedback.submit`, `workspace.feedback.list`, and `workspace.feedback.delete` through the same tools run path as every other capability. Which pack owns those tools is not decided in this skill. Run records use `runs_get`, `runs_events`, and `runs_cancel`, without a pack.

## The invariant

One report per task. List open reports first and pass `same_as` when one already matches. Never put a lead, prospect, or customer personal data in any field. Free text (`task`, `tried`, `error`, `suggestion`) is sent only when `consent_free_text` is true, and only after the user agrees. Otherwise send the structured fields alone.

## Choose one question

| What happened | kind | impact |
|---|---|---|
| The capability you needed does not exist | missing_capability | blocked, or worked_around if another tool finished the job |
| A tool failed | broken | blocked or worked_around |
| A tool or its repair was unclear | confusing | friction, blocked, or worked_around |
| You have an improvement and the task still finished | idea | friction |

Server-captured kinds (refusal, error, budget exhausted, abandoned dry run) are not filed from here.

## Evidence order

1. Finish the task, or stop because you cannot.
2. Call `workspace.feedback.list` and read the open reports.
3. If one matches this report, call `workspace.feedback.submit` with `same_as` set to that report.
4. Send structured fields you have: `capability_id`, `error_code`, `run_id`, `trace_id`, `kind`, `impact`.
5. Ask before any free text. With a yes, pass `--consent` so `consent_free_text` is true and include `task`, `tried`, `error`, and `suggestion`. With a no, omit those four fields.
6. Keep the returned delete token. `dreamstate feedback delete <report_id> --delete-token <token>` calls `workspace.feedback.delete`.

The CLI flags are `--kind`, `--impact`, `--capability`, `--error-code`, `--run-id`, `--trace-id`, `--same-as`, `--task`, `--tried`, `--error`, `--suggestion`, and `--consent`.

## Common findings

- **Missing capability:** the id you searched for was not in the catalog. File `missing_capability` with that id.
- **Broken tool:** the call returned an error code. File `broken` with the code and the run or trace id.
- **Confusing tool:** the schema or the repair did not say what to do next. File `confusing`.
- **Workaround:** another tool finished the job. File the original gap with impact `worked_around`.

## Guardrails

- Call `workspace.feedback.submit` once per task. Do not file again for the same gap.
- Do not submit while diagnosing. Hand the identifiers to this skill after the question is answered.
- Do not put a person's name, email, phone, profile, message, or any other customer data in any field, including free text.
- Do not pass `--task`, `--tried`, `--error`, or `--suggestion` without `--consent`.
- Do not invent a `same_as` value. Use one only when list showed a matching open report.

## What to return

- Whether a report was filed, its report id, and whether it joined an existing theme.
- The delete token, so the report can be removed.
- If you did not file, say why (no consent for the only details you had, or the task never hit a gap).

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule. Filing a report is free and needs no approval. Free text still needs the person's consent.
