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

## Choose the path

| Need | Path |
| --- | --- |
| Count, find or update durable records, notes, tasks or pipeline | Use CRM record and task capabilities. Search first, and inspect the model and identity fields before writing. |
| Enrich working lists or capture new signals | Use [`list-building`]; worksheets are working data, not durable records. |
| Import a CRM list or filter into a worksheet | Use the worksheet import path and synchronize do-not-contact first. |
| Import a whole object or keep a connected CRM in sync | Use the integrations connection and sync path below. |
| Promote worksheet data to records | Bind identities, then attach selected columns as described below. |
| Read or reply to a conversation | Use [`inbox-replies`]. |

A CRM list is a worksheet. Create, rename, archive, and delete it with `tables.worksheets.create`, `tables.worksheets.update`, `tables.worksheets.archive`, and `tables.worksheets.delete` (or `tables.workbooks.create` when the workbook is missing). Deleting a list never deletes the records. Membership writes are `crm.records.list_add` and `crm.records.list_remove`. Read them with `crm.records.lists_get` or `crm.records.memberships_get`.

Sample records stay out of customer totals. A current true value on `_sample`, `sample`, or `is_sample`, or an email only on a reserved example domain, marks a sample. `crm.records.keep` clears the sample flag on one record after a dry run, and only when the user asked to treat that record as real. Linked records stay flagged. Detail, the import, the bind-and-create path, and the sync ladder are in [crm-operations.md](crm-operations.md).

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

Field definitions include `wired_columns` and `latest_record_projections` when you pass `include_wiring: true`. Check target, status and skip reason when a field is empty. The bounded queue read can omit outcomes, so absence of an outcome does not establish success. A manual value, empty source or write policy can prevent a projection.

Use `senders.suppressions.exclusions_list`, `senders.suppressions.exclusions_preview` and `senders.suppressions.exclusions_explain` to inspect outreach exclusions before using the corresponding create, update or delete capability. Account limits use `senders.suppressions.exclusions_account_limits_get` and `senders.suppressions.exclusions_account_limits_set`. These capabilities belong to the senders pack.

Workspace webhook management is unavailable to model-driven calls. The workspace pack describes `workspace.webhooks.create`, `workspace.webhooks.update` and `workspace.webhooks.rotate_secret` for permitted surfaces; updates replace supplied filters, and rotation returns the new secret once with an overlap for the previous signature.

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

## Scoring

Scoring is deterministic and free. The rubric defines firmographic criteria with an operator, value and weight, each marked Required or Nice to have; it also defines A to D grade cutoffs and signal types with weight, half-life and lookback. Fit is A to F: a failed Required criterion gives F, and known Nice to have criteria determine A to D. Unknown values are not failures; when no criterion is known, the result is No data. Signal level is Burning, Hot, Warm or Cold, and priority ranges from 0 to 100. Explain fit, timing and priority separately; do not represent priority as fit. Customers, competitors and rubric exclusions carry exclusion labels. Deals use the derived company-first score.

Create or change the rubric only at setup or when the user asks. Read `crm.score_rubric.get` and field definitions with `crm.record_objects.attributes_list`; draft criteria from evidence and resolve missing decisions. Call `crm.score_rubric.set` with `{ "rubric": {...}, "expected_revision": <revision>, "apply": false }` to preview grade distribution and coverage. Review the result, then apply the intended rubric with `apply: true`; this saves a revision and queues a free rescore. Use `crm.scores.recompute` for a record or the workspace when needed. Use `crm.records.score_explain` to show criteria, signals, timestamps and whether the stored score is stale. If the revision conflicts, read again and reconcile instead of overwriting another change.

## Signals

Read stored activity with `crm.records.activity_timeline_get` and rank today's candidates with `crm.leads.today_list`, which uses recent replies, comments and other engagement with stored fit evidence. Explain the signal and its freshness; a missing fit score is not zero. To investigate public LinkedIn post signals, `crm.signals.search_filters` reads available filters, `crm.signals.search_plan` plans routes and costs, and `crm.signals.search_preview` reaches the provider for a free preview that consumes search budget. `crm.signals.search_run` sources onto a worksheet: inspect its dry-run quote, pilot a bounded scope, then follow the paid-run consent rule. Signal capture belongs to worksheets, and automatic enrollment belongs to sequences. Do not infer complete coverage from an empty or partial source.

## Bind and promote worksheet data

This is deliberate CRM work, not a prerequisite for outreach. Clean malformed identities and inspect duplicates first. Preview `crm.records.unbound_rows_enroll` with a specific `table_id` and bounded `limit`; a null table id covers all eligible worksheets. Review would-bind, would-create, ambiguous and unusable rows. Live enrollment binds or creates people, a truth write; it does not create companies or write worksheet cells. Inspect failures and remaining rows before continuing.

After rows are bound, attach an explicitly chosen worksheet column to a CRM attribute with `tables.columns.crm_target_attach`. This queues completed cells and keeps later updates flowing, so inspect the target, existing values, write policy and backfill scope first. The default fills blanks; the alternative preserves manually entered values. Preserve identity fields and never overwrite identity from worksheet data. Check wiring, skipped outcomes and record values after the backfill; a missing queue outcome is not proof of success.

## Example: today's follow-up

1. Run `crm.leads.today_list` with `{ "limit": 10, "since_hours": 72 }`. Read ranked people, signal timestamps, reasons and nullable fit. Choose a candidate supported by fresh evidence.
2. Resolve the person with `crm.records.search` using `{ "q": "<verified email or name>", "object_key": "person" }`; do not guess if matches are ambiguous. Run `crm.records.score_explain` with `{ "record_id": "<person UUID>" }` and read its criterion and signal explanations and freshness.
3. Create a reviewable follow-up with `tasks.create` and `{ "title": "Follow up with <name>", "type": "follow_up", "status": "draft", "links": [{ "link_kind": "record", "link_id": "<person UUID>" }] }`. Add a due date or assignee only when known. Read the returned task and link, then verify with `tasks.get` using `{ "task_id": "<returned task UUID>" }`. Report the evidence, task and record links; a draft does not notify its assignee.

## CRM connections (HubSpot, Attio, Salesforce)

Inspect connection health and permissions first. If access is missing or a connection is unhealthy, stop and report the reconnect or permission requirement; do not bypass the connection.

1. List connections with `integrations.outreach_connectors_list` and take `connection_id`.
2. Read settings with `integrations.crm_management_get`.
3. Read available fields and granted permissions with `integrations.crm_management_metadata`.
4. Count before changing with `integrations.crm_management_preview`.
5. Review the current dry-run quote, scope, row limit and credit ceiling under the paid-run consent rule before enabling a live import or sync. Start with a small pilot and inspect its outcome.
6. Change import, push, or mappings with `integrations.crm_management_save`. Pass `expected_version` from the get call.
7. Run now with `integrations.crm_management_refresh`.
8. Read failures with `integrations.crm_management_logs` filtered to result `failed`.
9. Retry one record with `integrations.crm_record_retry`.

These tools belong to the `integrations` pack and are also listed in the `crm` pack. `crm.sync.*` schedules one object pair; `integrations.crm_management_*` holds the import rules, push rules, and field mappings.

Sync rules:
- For list or filter imports, synchronize the do-not-contact source before any rows land. Abort the import if that synchronization fails. Skip it only on an explicit user instruction, and make the omission visible. Suppression is additive; remove suppression by subject only after confirming intent.
- Imported provider-supplied fields are last-write-wins and can replace local edits. Review field mappings and their update rules before running; do not assume a field-level merge. Bound row volume and spend independently, including for recurring runs.
- A record deleted in the CRM is not deleted here. It is marked so it is not imported again or pushed back.
- A contact imports even when the CRM did not grant company permission; it arrives without a company link.
- A record that matches more than one existing record is skipped and logged, never guessed. Merge the duplicates, then retry that record.
- Each field mapping carries its own update rule. The defaults a new connection gets: a value imported from the CRM replaces ours, and a value pushed to the CRM only fills what is empty there.
- Push creates people in the CRM only for the audience the settings name (by default, people who replied).

`crm.sync.status`, `crm.sync.configure`, `crm.sync.enable`, and `crm.sync.disable` are the object-pair schedule. Read status first. Preview configure and enable in dry run. A live enable follows the paid-run consent rule, including its row ceiling, credit ceiling, and any approval, then read status again. Enable and disable accept HubSpot (`contacts`, `companies`) or Attio (`people`, `companies`) only. Configure previews `company`, `person`, or `deal` and returns `next_cycle` with `saved` false on a dry run. Disable stops the cron and keeps mappings. The worked params and readback are in [crm-operations.md](crm-operations.md).

## Default deal stages

- Lead: A deal exists after a positive reply or manual creation.
- Meeting Booked: A meeting is booked or rescheduled.
- In Progress: The meeting happened or the deal was qualified.
- Won: The deal was won.
- Lost: The deal was lost.

## Credit posture and sensitive effects

Native record reads, duplicate review, and timeline inspection are free and come before enrichment. External CRM calls can have per-call costs, including discovery and pagination; inspect the current quote and provider quota. A stored quote is a snapshot, so refresh it before a live run. For paid enrichment, state target records, fields sought, expected coverage, credit estimate, and how result will be reviewed, then run it under the paid-run consent rule. Do not enrich records merely to fill blanks.

Merging records, changing consequential pipeline state, bulk edits, or any action that may hide human judgment needs evidence and exact change shown first. Then act when the user asked or confirms. Keep adjacent cleanup separate.

## Guardrails

- Read exact schemas and object definitions before writes. Unknown keys, missing dropdown choices or an object without a usable identity can reject writes or create duplicates on re-import. Use bounded record reads and inspect omitted details when needed.
- Treat any marked sample records separately from real customer totals and ask before treating them as real.
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
- Record, run and worksheet links for changes when available.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
