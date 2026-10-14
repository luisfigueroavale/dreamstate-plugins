---
name: first-win
description: "Run a short first session in a new workspace that confirms the user and workspace, builds or reads Company Context, creates one small sampled worksheet from the ideal customer, and suggests three next plays. Use it on a fresh workspace, when the user says \"get me started\", or when nothing has been built yet. It does not launch outreach, publish content or run anything at scale."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. A larger paid run also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# First Win

## Boundary

- Owns: first session with accurate context and a small qualified sample.
- Picking and running the next named play: [`plays`].
- The `tables` pack lists the tools and their exact inputs.

Start a first-win request with one search for the first-win bundle, limit 10 and a 30-day window. It returns workspace offer, ICP and voice excerpts with document ids; ranked warm leads; labelled zero-credit cold preview only when warm leads are short; top three blueprint recommendations; exact unexecuted next calls. Do not split this discovery into separate reads unless returned bundle names a missing or failed source.

## Backing packs

The knowledge pack backs Company Context. CRM, Unibox, LinkedIn signals and existing worksheets supply warmth evidence before anything cold. Rank interested unanswered inbox threads and engaged people before cold search, using contact name, company and profile URL from list rows. The tables pack backs first-leads worksheet and its fallback source. Start with its free provider preview, capped at ten rows and zero credits. Read filter coverage: only applied required filters qualify a row; unverified or unsupported required filters mean possible fit, never qualified. The workspace pack reports credits and action costs when the user asks what sample will cost. Everything else waits for later play.

## The invariant

Build real things, keep the first spend small, and send nothing. The session ends with saved pages and a worksheet the user can open, not a description of what could be done.

## Practice

1. **First-win bundle.** Search for the first-win bundle first, with limit 10 and a 30-day window. It concurrently reads workspace context, warm evidence and blueprint state. It merges open CRM deal contacts, interested/question/objection unanswered inbound replies, and recent stored LinkedIn post engagers; it removes suppressed people and returns ranked evidence. Read individual sources only to inspect a returned row. Rank evidence by rung: active relationship, interested unanswered reply, recent CRM evidence, LinkedIn engagement, then existing qualified row. Never claim a warm rung without returned evidence.
2. **Confirm identity.** Use whoami only when bundle workspace identity conflicts with user request. With more than one workspace, confirm which before writing anything.
3. **Company Context.** Load knowledge skill. If bundle has missing offer or ICP, build Product Information, Ideal Customer, Competitor Analysis and Tone of Voice by reading company website now and searching competitors, read them back, show Ideal Customer. Ask one question only if ICP has two readings that land different people.
5. **First leads.** After warmth reads, install the list blueprint the bundle recommends (it creates the worksheet, its source and the Qualified view), configure its source inputs, or reuse an existing worksheet. Land about ten people, ranking warm people before fallback matches. Every row needs warmth rung, one evidence reason, one Ideal Customer fit reason, and drafted first message. Drafts stay internal. Do not look up contact data, send, enroll, or start a sequence.
6. **Cold fallback only when needed.** Load list-building skill only when warmth ladder cannot fill set. Search source cards for people or companies matching scarcest Ideal Customer criterion. Start with free preview, state zero-credit result and ten-row ceiling, then run at most about ten fallback rows. For every required filter, inspect coverage before ranking: applied can support fit, unverified or unsupported cannot. Do not run paid search before dry run.
7. **Rank and draft.** Add Required criteria, score, evidence and first-message draft columns. Price any scoring or draft run. Keep session at zero credits when warm rows suffice; otherwise use only one dry-run-approved sample ceiling. Read ranked rows back, show about ten with reason and draft, and report credits charged.
8. **Adjust once.** If fewer than half fallback rows look right, change source filter and repeat dry run before another bounded sample. Do not add columns to hide poor source fit.
9. **Suggest three plays.** After first-leads readback, search for the read that recommends three blueprints this workspace can run now. Present returned top three, including `why_now`, unmet prerequisites, credit posture and exact describe call. Load plays skill for operating guidance, then recommend one returned blueprint and say why. Never substitute static catalog order for this workspace-state read.

Search the tool for each task. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run.

## Guardrails

- No sequence starts, no posts published, no emails or DMs sent in this session.
- No contact data lookups yet. They come after the user agrees the sample is the right people.
- Warm reads and free preview come before every cold source run. Keep paid work to one bounded sample, or zero credits when warm rows suffice. Show coverage, preview and estimate, then run the bounded sample from that preview. Anything larger follows the paid-run consent rule.
- A missing account never blocks building. Build the worksheet, then name the account to connect.
- Do not start a play on the user's behalf. Recommend one and let them pick.

## What to return

- Workspace confirmed.
- Company Context pages created or updated, with revisions, and any open question.
- Worksheet link, warmth ladder results, source and filters, criterion columns (Required or Nice to have) and run conditions.
- First leads: about ten ranked people, warmth rung, evidence reason, fit reason, first-message draft, credits charged.
- Three suggested plays, one recommended, and what each needs.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
