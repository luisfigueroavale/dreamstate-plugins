---
name: dreamstate
description: "Use Dreamstate for sales and marketing work: finding companies or people, enrichment and scoring, LinkedIn and email outreach sequences, inbox replies, LinkedIn and X posts, CRM records, workflows and AI search visibility."
---

# Dreamstate

Dreamstate is reached through its MCP server. Its tool list is the door, not the catalog: a small set of controls reaches every capability.

Prefer the narrower `dreamstate:*` skill for the job when one fits; `skills_get` returns the workspace's current copy of any skill.

1. `whoami` names the user and the active workspace. If it lists several workspaces, confirm the right one before writing anything.
2. `packs_route` maps a goal to a domain; `packs_get` returns that domain's capabilities and skills.
3. `tools_search` finds the capability for the goal; `tools_get` returns its exact input schema. Read the schema every time, never guess params from memory.
4. `tools_run` runs it with `capability_id` and `params`. The gating inputs (`mode`, `max_credits`, `idempotency_key`, `quote_hash`, `approved`) go on the run call itself, never inside `params`. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling.
5. `runs_get` follows long work; `runs_events` and `runs_list` read more of it and `runs_cancel` stops a run.

Lists, finds, enrichments and signals happen only inside a workbook worksheet. Report only what tools returned.

## Paid and outward-facing runs

For writes, credit spend or external effects: dry_run first, review the row count and credit estimate, then run live with the quote_hash that dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. Supply max_rows or max_live_sends only when the fetched capability schema declares them; when an external send accepts max_live_sends, cover at least the dry_run recipient_count. Sequence lifecycle and enrollment calls have no lifetime send ceiling. A quote returned by a refused live call is only a price, never a confirmation.

A paid run above the server's row or provider-spend limit, a live external send above the server's recipient threshold, or a run that turns on a recurring trigger, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply.

`max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices.
