---
name: first-win
description: "Older name for the setup skill. Load setup instead."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# First Win

## Boundary

- Owns: nothing. This id is kept so older installs and links keep working.
- First session setup: [`setup`].
- The `knowledge` pack lists the tools and their exact inputs.

First Win is now the `setup` skill. Load `setup` with `skills_get` and follow it: it confirms the workspace, writes Company Context and the score rubric, builds a first workbook, drafts a first sequence and hands over account links.

Paid and outward runs follow the paid-run consent rule.
