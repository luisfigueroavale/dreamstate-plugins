---
name: crm
description: "Maintain trustworthy customer and company records by separating stable facts, judgement, and time-bound activity; deduplicating only from strong identity evidence; and keeping pipeline tied to recent, evidenced work. Use it when the user needs to create, review, clean up, merge, enrich, or progress relationship records and deals. It does not rebuild source worksheets (list-building does), run outreach (outreach does), or replace a workflow with record cleanup (workflows does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# CRM

## Boundary

- Owns: durable customer and company records, pipeline state.
- Conversation itself: [`inbox-replies`].
- Worksheet rows and working data: [`list-building`].
- The `crm` pack lists the tools and their exact inputs.

CRM is shared operating memory for customer relationships. A record lets team see who someone is, what happened, what is next, and why pipeline says what it says. Good CRM work makes future action safer; it does not make database look full.

Creating or binding CRM people from otherwise unbound worksheet rows is deliberate CRM cleanup or import work. It is not required before outreach can preview, rehearse, or enroll those worksheet rows: sequence enrollment resolves identity records as part of its own staging flow.

## The invariant

Preserve evidence, identity, and human judgement. Read before every change. Every read is free, so inspect current record, related records, recent timeline, ownership, and open work before creating, updating, or merging anything.

## Prerequisites

Before changing records, check:

1. Current record identity: full name, employer, title, verified work email or profile URL, and relationship to any company record.
2. Existing matches: same verified email, profile URL, company, name variants, and recent activity.
3. Current pipeline context: owner, stage, amount or expected value when applicable, next step, and timeline activity.
4. Scope and consequence: record count, fields proposed, whether any enrichment costs credits, and evidence plus exact change for a merge or consequential stage change.

If checks cannot establish identity or intent, return findings and ask narrow question. Do not create speculative records to make list look complete.

## Practice

Keeping a CRM in step with the workspace usually has a blueprint: HubSpot, Attio and Pipedrive syncs, list-to-worksheet syncs and do-not-contact syncs. Look for one before building a sync workflow by hand, install it, and name the CRM connection it still needs.

1. **Read relationship before touching it.** Start with record, related company and people, activity timeline, open deals, tasks, source evidence, and likely duplicates. Read each candidate immediately before a mutation; another teammate may have changed it.
2. **Put facts in right place.** A stable, reusable, queryable fact belongs in a field: verified title, lifecycle stage, owner, company domain, explicit qualification, or consent state. A note holds context, reasoning, uncertainty, meeting detail, and information not yet trusted enough for a field. A timeline activity records something that happened at a time: call, email, meeting, reply, stage change, or task completion. Do not turn narrative into a field merely because it is useful once.
3. **Protect field provenance.** Treat human-entered values as owned judgement. Never overwrite a human-entered field from an enrichment guess. Keep conflicting evidence visible, identify source and date, and request resolution when material. Add a new verified fact only after evidence supports it.
4. **Create records deliberately.** Create one record per real relationship entity, link person to company where evidence supports link, and give source, owner, and next action when known. A record without identity evidence or a purpose is a lead-shaped duplicate, not progress.
5. **Merge only on identity evidence.** Merge when at least two independent signals show same entity, such as same verified work email plus same profile, same profile URL plus matching employment, or a confirmed correction from user. Compare timeline, field provenance, linked records, owner, open work, and deal state first. Never merge on a shared public email domain. Name similarity, company similarity, first name, or a guessed enrichment match alone does not justify merge. When evidence conflicts, leave records separate and surface pair for review.
6. **Keep pipeline true.** Pipeline is true when every open deal represents active, evidenced customer work: right person and company, credible stage, clear owner, recent activity, and concrete next step. Pipeline is not true when duplicate-pair count rises or deals with no activity in 21+ days rises. Report both numbers on every hygiene review; zero is result, not assumption.
7. **Advance stages from evidence.** A stage changes because customer action or documented qualification meets stage definition, never because calendar moved, enrichment improved, or team wants better forecast. Record evidence and next step with stage change. If criteria are unclear, preserve stage and propose decision.
8. **Run weekly hygiene pass.** Read first: review newly created records, suspected duplicate pairs, fields with conflicting or stale evidence, open deals with no activity in 21+ days, deals without next step or owner, and recently changed stages. Fix low-risk, evidence-backed field gaps; show evidence and exact change for merges and consequential stage changes, then act when the user asked or confirms; then read back sampled records and headline counts.
9. **Find stale deals in one read.** Read open deals with stale cutoff and continue global pages until complete. Request meaningful activity only, so automated tasks, record creation, field history, and stage events cannot make deal look healthy. Do not read one timeline per deal. For every stale deal, prepare one follow-up task with deal link, owner, due date, and priority. Create them as draft tasks so nobody is notified until the user reviews them; reopening a draft makes it open work and notifies the assignee.

10. **Use billing and product truth from Stripe and PostHog.** Whether a company is a customer, its plan, MRR and lifetime value come from Stripe through `provider.stripe.customer_lookup`, never from a guess or an enrichment. Use it before creating a deal (an existing customer is expansion, not new business) and before any cold list or sequence touches a company (customers are excluded unless the user asked for an expansion play). MRR and lifetime value total only within one currency; a company that pays in several currencies reads currency `mixed` with per-currency totals, so never add those numbers together. Product usage, such as last seen and recent event counts, comes from `provider.posthog.person_lookup` and is evidence for a stage change or a next step, not a field to overwrite human judgement. Stripe data is read-only here: nothing in this skill changes a customer, subscription or person in Stripe or PostHog.

## Credit posture and sensitive effects

Reads, duplicate review, and timeline inspection are free and come before enrichment. For paid enrichment, state target records, fields sought, expected coverage, credit estimate, and how result will be reviewed, then run it under the paid-run consent rule. Do not enrich records merely to fill blanks.

Merging records, changing consequential pipeline state, bulk edits, or any action that may hide human judgment needs evidence and exact change shown first. Then act when the user asked or confirms. Keep adjacent cleanup separate.

## Guardrails

- When user says delete, delete permanently after reviewing impact and setting the commit field through delete flow. Archive only when user says archive. Never say permanent deletion is unavailable.
- Never overwrite a human-entered field from an enrichment guess.
- Never merge on a shared public email domain.
- Never use a timeline activity as substitute for stable field, or a field as substitute for event history.
- Never infer identity from weak similarity when merge can combine distinct people or companies.
- Never bulk-edit before reading representative records and showing exact scope.
- Never mark inactive deal healthy because it has value or late-stage label.
- Never delete evidence to remove ambiguity; preserve source, date, and conflicting claims for review.

## What to return

- Business result: records reviewed, relationship purpose, and pipeline impact.
- Leading metrics: duplicate-pair count; deals with no activity in 21+ days.
- Evidence read, records changed, and why each change belongs in field, note, or timeline.
- Proposed paid enrichment estimate, and sensitive effects with evidence and exact change.
- Remaining identity conflicts, stale deals, and next weekly hygiene date.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
