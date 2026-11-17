---
name: list-building
description: "Build a workbook of companies or people: as few sources as possible, people at companies through the employees enrichment fanned out to a second worksheet, one column per criterion marked Required or Nice to have, run conditions that skip rows already failing, an optional score, sample then scale, and honest coverage."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Workbook building

## Boundary

Owns: sources, columns, criteria, and building qualified lists.
- Multi-step outreach sequences: [`outreach`].
- Existing worksheet structure and values: [`worksheet-tidy`]. Waterfall columns and one-record enrichment: [`enrichment`].
- Durable customer and company records: [`crm`]. Warm sources: [`linkedin-warm-signals`].
- What past outreach produced for an audience: [`performance`].
- Explaining cell values and failed runs: [`diagnostics`].
The `tables` pack lists the tools and their exact inputs. When user says delete, permanently delete named workbook, worksheet, template, automation, contact, row, or column after reviewing removal impact and setting the commit field. When user says archive, archive only. Never say permanent deletion is unavailable.

A list is worth contacting when every row would recognise itself in the reason you contact it. Find the smallest set where that is true, prove it on each row, and stop.

## Pick the path

| Ask | Where the rows come from | First calls |
| --- | --- | --- |
| Company list: firmographics, a TAM, lookalikes, hiring companies | One company worksheet and one company source | `tables.executable_definitions.list_resolved`, then `tables.sourcing.filters`, then `tables.sourcing.preview` |
| People list: a role, a place, or people at named companies | A people source only when that definition declares every Required company fact. Otherwise companies first, then `tables.worksheets.create_fan_out_destination` | Same preview, on the source you will actually attach |
| Signal list: hiring, funding, post engagers, followers, profile viewers, X posts | One worksheet source. Capture stays here. A score or today's leads belong to crm. Auto-enroll belongs to outreach | Exact blueprint via `workspace.blueprints.recommend`, or the ready definition from `tables.executable_definitions.list_resolved` |
| Import: the user already has the rows | CSV or a URL of rows. Not a search | `tables.table_sources.csv_import_prepare` or `tables.table_sources.url_import_preview` |

Do not use this skill to enrich rows you already have ([`enrichment`]), to send ([`outreach`]), or to merge records (crm). A signal list is not a cold people search. An import is not a preview of a provider.

Source choice, in this order:

1. Import when the rows already exist. Verify the staged count with `tables.rows.query` before any paid column.
2. A signal source when the hardest Required criterion is attention or a public event. One source on the worksheet. Install a blueprint only when it is an exact match, including every Required filter.
3. A people source when `tables.executable_definitions.list_resolved` returns a ready definition that declares every Required person fact and every Required company fact.
4. Companies first when the people source does not declare headcount or industry. Qualify the companies, then fan out.
5. Stop after one empty `tables.sourcing.preview` that you have already loosened once. A second source is not a way to hide an empty preview.

Sample size:

- `tables.sourcing.preview` is the quality check. A target or a sample length is not a market size. Quote a count only when that preview returns one.
- The user named a target. The first paid run is a sample small enough to read row by row. Scale the same source in bounded batches toward the target. Over-provision about 1.4 times the target only on that scale step, and only because later criteria drop rows.
- The user named no target. The first paid sample is about 25. Read every row. Ask how far to grow only after that sample settles.
- Under about 30 percent qualified, name the Required criterion that fails most and propose one change. Under about 10 percent, change the source.
- A preview that returns nothing: loosen one filter and preview again. If it is still empty, the source cannot express the request.

## Rows and qualified rows

- One workbook per list; each worksheet holds one kind of row and at most one live source. Everything else is a column.
- A row is a raw entry from the source. A qualified row passes every Required criterion.
- Track every criterion the user named, even when the source already filtered on it, and mark it Required or Nice to have. A criterion is a condition, not a column: put it at the cheapest place that decides it (a source filter, a run condition on a value already on the row, or a condition in the fit score). Its result is pass, fail or unsure. Missing data is unsure, never a fail.
- Keep disqualified rows: they are paid for. Views decide what to show.
- Before building a list, read the workspace exclusions with `senders.suppressions.exclusions_list`. If there is no Competitors exclusion and Company Context or the request names competitors, propose creating a company exclusion named "Competitors" (`senders.suppressions.exclusions_create`, `record_type` company, condition `domain` `is_any_of` the competitor domains) and say that exclusions apply to every sequence on the chosen channels. Competitors and internal team belong in workspace exclusions, not in per-list run conditions. When exclusion is required and Company Context plus request contain no competitor values, ask once for competitor names before any paid sample. Do not source approximate list, create paid rows, or substitute keyword exclusion.

## Blueprint or build

Before building, see if a blueprint does exactly what you need: its source is the right source for the hardest criterion and its columns are the chain you would build. If one does, install it, then configure its columns. If not, build your own workbook. Never install a near miss to have something to start from.

When asking for a blueprint recommendation, pass `required_filters` for every user-stated hard constraint, using its closed filter enum. Read `unmet_required_filters`; do not install a recommendation that cannot apply every Required filter.

For people engaging with competitor company posts, `signal-engaged-company-posts` exact match. Prefer it. It sets `max_posts_per_company=2`, `max_engagers_per_post=25`, `sample_size=25`, then excludes employees of selected companies in first paid qualification. Attach the company engager source directly only when a custom workbook is needed: send `company_urls` as literal string array, keep same caps, add employee-exclusion qualification before outreach.

For incoming messages or booked meetings, read Incoming people sources (`incoming-people.md`). Use `signal-received-dm` for Received DM plus AI fit score, or `signal-meeting-booked` for Meeting booked plus profile enrichment. Each installs exactly one people worksheet and two columns. Qualification follows the user's rubric, which may describe sales pitches, support requests, or another purpose. These sources do not require the general prospecting enrichment and email chain below.

Signal capture is a worksheet source. Post engagers, followers, profile viewers, hiring searches, funding searches, and profile watchers land as rows on a worksheet in this skill. Reading a score, a signal level, or today's leads belongs to crm. Auto-enroll belongs to outreach. Name a public signal in cold copy (hiring, funding, a public post, a job change). Never name a private one (a profile view, a website visit, who engaged privately).

A first session in a workspace where nothing is built yet belongs to the setup skill. The id first-win only aliases setup. This skill starts when the user asks for a list, including the sampled list that setup requests.

## Customer and product sources (Stripe, PostHog)

Your own customers and product users are lists too. Use them when the request is about expansion, upsell, win-back, product-qualified leads, or keeping customers out of a cold list.

- **Stripe customers.** `source.stripe_customers` lands one person row per Stripe customer from the workspace's Stripe copy, keyed by `stripe_customer_id`. Its filters are `statuses` (`active`, `trialing`, `past_due`, `canceled`, `unpaid`), `price_ids`, `plan_names`, `created_after`, `created_before`, `min_mrr` and `max_rows`; put every billing criterion there, not in a later column. Outputs include plan, subscription status, monthly MRR, lifetime value, first paid date, trial end and delinquent. Use it for expansion lists (active on a lower plan, MRR above a floor), trial conversion (trialing, trial ending soon) and win-back (canceled).
- **Stripe lookup column.** `provider.stripe.customer_lookup` answers "is this row already a customer" from an email or company domain column, with `is_customer`, plan, status, MRR, lifetime value, first paid and canceled dates. Add it to any cold list as an exclusion: a free column whose run condition is that the email or domain is set, then make `is_customer` `neq` true a Required criterion on the first paid column and on enrollment. A free-mail domain never matches a company.
- **PostHog cohort.** `source.posthog_cohort` lands the people in one PostHog cohort, keyed by `distinct_id`, with `signal_source_id`, `cohort_id` and `max_rows`. List the project's cohorts first and pick by name; never guess a cohort id. Use it for product-qualified lists when the cohort already states the behaviour the user means.
- **PostHog lookup column.** `provider.posthog.person_lookup` adds last seen, 30 day event count, top events, plan and properties to rows that have an email or company domain. Use its values in run conditions and in the fit score rubric as usage evidence (active in the last 7 days, used a named feature). For a one-off question about product data that no column answers, run a PostHog query; it reads the data without changing the worksheet.
- Both sources are monitor sources: after the first landing, new Stripe or PostHog events for a person update that person's row in place, and its Stripe and PostHog lookup cells refresh. Paid columns on the row still follow their own run conditions.
- Money: MRR and lifetime value are in major units and monthly. When a customer or domain pays in more than one currency, MRR and lifetime value are empty, currency reads `mixed`, and `revenue_by_currency_json` holds the totals per currency. Treat `mixed` as unsure, never as zero.
- A Stripe or PostHog source or column needs that account connected in the integrations pack. A missing connection blocks running, never building.

## Blitz sources and enrichment

Blitz enrichment products generally need a LinkedIn URL. When starting with a company domain, resolve it first with Company LinkedIn URL from domain. Blitz search stops at 50,000 results, so split larger searches by employee range, then region, then title. Preview the count for free before running a source.

## Free preview before build, complete chain before paid sample

Use the zero-credit source preview before workbook creation when the source permits a provider preview. That preview is a free read. Door modes `dry_run` and `live` both execute it and charge zero. Pass `definition_id`, `source_inputs` for that source contract, optional `table_id`, and `limit` from 1 to 10. Unknown top-level keys are rejected. It returns named sample rows, writes no worksheet, and charges zero credits. Do not attach a source merely to see the first rows.

Before first paid source run, configure every run condition, Required qualification column, AI fit-score column, and requested work-email preview column. Adding/configuring columns is free. First paid sample must land into chain already able to qualify source rows and gate later spend. Read every configured column back before source run. A source-run warning `qualification_columns_missing` means stop and configure the chain before spending.

## Configure every column before paid source runs

A column is configuration plus a run condition, set together when the column is created or configured, never added bare and fixed later. A source works the same way: its configuration is its filters. Every source and column schema shows its full configuration and run condition; set them there, before the first run.

- **Configuration:** the inputs that make it answer this request (source filters, titles, locations, prompt referencing `{{column}}` values and the user's criteria).
- **Run condition** (the column's run if setting): a filter on spend. It decides row by row whether the column runs. A view filter only hides rows and saves nothing.

Filter as early as possible. Put each criterion at the first place that can filter it:
1. The source filter, when the source has one for it (location, keywords, date posted). Source filters change what is fetched, so failing rows never arrive.
2. Otherwise, the run condition of the first column that produces the value, so every later paid column skips the row.
3. Only judgment the data cannot express goes into a scoring column.
Never leave a criterion source can filter to later column or score. Use source own filter fields: similar input elsewhere (scoring column size setting) filters nothing at source. Before first paid source run, read source back to confirm every filter is on it. Change source filters only before first paid run. Criterion source cannot fetch on goes into run condition of first paid column, on source row value, for example company headcount `gte` 50 and `lte` 200.
- Show source values with a formula referencing the source field. Never type a value into a column unless the user asks for a fixed value.

Run conditions as filters:
- Every column that costs credits or calls a provider has one.
- It names the fields that must hold first, with `is_set`, and excludes rows an earlier criterion failed: `eq`, `neq`, `in`, `not_in`, `contains`, `not_contains`, `gt`, `gte`, `lt`, `lte` on that column's value.
- Chain them: each paid column's condition includes every earlier Required criterion it depends on, so a disqualified row stops costing anything.
- Exclude rows that failed, not rows that are still unsure, unless the column is only worth running on qualified rows (contact data, emails): those require a pass.
- Order columns cheapest and most selective first: free formulas and exact fields, then enrichments, then judgment columns, then contact data.
- When a waterfall returns the data a column needs (work email, phone, people at a company), add the waterfall, not one of its single-provider steps: it tries every enabled provider in workspace order and stops at the first verified match. Use a single provider only when the user names it or the waterfall cannot take the row's inputs, and say which.
- Read the column back after saving and confirm the condition is there.
- Research column shape, fixed output: read `column-authoring.md`, Research columns.

Disqualifiers and the fit score:
- A disqualifier the row already decides is a run condition, never a new column. Role or headline keywords: one `not_contains` condition per word (student, intern, university) on the title or headline. Size and dates: numeric ranges. Put those list-specific disqualifiers on the first paid column and on the enrollment gate. Competitors and internal team belong in workspace exclusions, not in per-list run conditions.
- To add or change list-specific disqualifiers on an existing worksheet, edit the run conditions of its existing columns. Do not add columns for them.
- The AI fit score is one `column.ai_fit_score` column (TypeSafe, numeric `score` 0 to 100), never `column.ai`. column.ai_fit_score is the way to score fit, ICP, or lead qualification, and it is the cheaper option. Its rubric lists every criterion as Required or Nice to have, with the row's signals (engagement, hiring, recency, role) as evidence. A Required fail scores 0; missing data is unsure. Views and enrollment read the numeric `fit_score_value` (the `score` field of `fit_score`), not the score object. Enroll only rows at or above the threshold the user set.
- Add an AI or research column only for judgment the row cannot express (what a company actually sells, whether a post is a real complaint), never for something a keyword or a list match decides.

## Loop

1. **Discover.** From the request, the conversation and saved context: what the list is for, what qualifies and disqualifies a row, Required versus Nice to have, and the narrow or broad reading of vague terms. Ask only what is still missing. When you need values (URLs, names, numbers), ask for the values themselves as a free-text answer; options are possible answers, never ways of answering. Never ask how many up front.
2. **Blueprint or build.** Use a blueprint only if it does exactly what you need; otherwise build your own.
3. **Hardest criterion first.** Select source that natively owns every Required company fact before convenient connected-account search. Search resolved executable definitions for ready people-source definitions and their exact input contracts. Select the returned hard-firmographic people source when it owns person location, company size, and industry, then attach that source. If no resolved people source owns hard company facts, source companies first with the resolved company search using exact company size and industry, then fan out people from qualified companies. The resolved LinkedIn people search owns role and LinkedIn facets but does not declare company headcount. Never replace missing size or industry with `SaaS` keyword. Use LinkedIn source only when declared facets cover every Required pre-spend fact, or after company-first qualification. If LinkedIn provider contract later exposes company headcount, use declared facet; never claim it exists before contract says so.
4. **Free source preview, then configure chain.** Call the zero-credit source preview first when available. Read named rows against every criterion. Aim for more than half right at source. If source can filter failing criterion, fix input and preview again; if it returns nothing, find over-narrow input. If source cannot express hardest criterion, switch source now, before workbook attachment or spend. After free preview, build complete Required qualification, 0 to 100 fit-score, and work-email preview chain with run conditions.
5. **Paid sample only after chain exists.** Run bounded paid source sample, then run configured chain on sample and read cells. Fix and rerun until correct. Follow every run to end.
6. **Audit.** When zero rows qualify and near misses show stated-role equivalents, widen once inside stated intent, resample, and say what you widened. Report qualified of total and cost per qualified row. Under about 30 percent qualified, name the Required criterion failing most and propose one change; under about 10 percent, propose a different source. Then ask how far to grow the list.
7. **Scale the same source** in bounded batches toward the target, running paid columns only on rows their conditions admit. Only qualified rows count. Call a source exhausted only when a run returned fewer rows than it asked for.

To grow an existing list, rerun its source with a higher target and widen only with levers that keep every Required criterion; report near matches separately.

## Verification

### X posts to verified people

Use the direct X source definitions returned by the catalog. Search posts for a topic or complaint; use profile-post sources for supplied accounts and post-URL sources for supplied posts. Discover the exact profile and post engager contracts before configuring them. An app-only search credential does not imply access to likes or other endpoints that require a connected X account.

Keep one row per post in the source worksheet, with the post ID, author identity, biography, location and website available as source outputs. A post and its author are different identities: do not collapse a person's multiple posts before qualifying their meaning. Keep original source payload and provenance.

For a genuine-complaint check, use an AI column over the post and author data with an explicit boolean result and a short evidence-based explanation. Gate later paid research on the qualification result. Configure the complete chain before running a small paid sample.

Use `column.web_agent` for identity research; [x-identity-research.md](x-identity-research.md) gives the reusable prompt, output declaration and sample review. Configure its Outputs using the same output contract as other AI columns, with a nullable LinkedIn URL, confidence, concise match explanation and matching evidence. Bind name, handle, bio, location and website from the source using column references. Select available search tools explicitly; do not silently switch providers. Search for candidate public LinkedIn profiles and compare company, location and other independent signals. A shared name alone is insufficient. Return a URL only when it was observed in retrieved evidence and the identity is supported. Ambiguous or missing matches remain unknown. Never fabricate a URL, cite an unobserved page, or treat a confident explanation as proof.

Establish whether the LinkedIn match is Required or Nice to have from the request before scaling. A missing match is unsure, not a failed qualification. Contact enrichment runs only when requested and when the verified profile URL is set; bind that output into the existing enrichment column rather than copying it into a manual cell.

Read the sample's actual cells, citations, failures and credits before growing the source demand. Increasing demand resumes its saved pagination and paid page tail. Monitoring new posts and continuing older pages are different operations; use the source's declared controls. A fulfilled demand should not trigger another provider read. Do not promise that refreshing mutable engager lists is free or that a source is exhausted merely because one page contains duplicates.

For problem-mention searches, derive the problem and intended audience from the request and Company Context. Do not silently narrow a general complaint request to acquisition, cold email, or another use case. Inspect a small source sample before expanding. If it produces no qualified authors or mostly unrelated meanings, review the query and qualification rubric before buying more pages. Check uncertain decisions against the complete post; a truncated post is incomplete evidence. Do not exclude replies or choose recency versus relevance without considering where the requested signal occurs.

Report source rows, qualified authors, authors actually researched, candidate profiles and confirmed matches separately, alongside the credits for each stage. A candidate in a summary is not a confirmed match. Declare candidate URL, confirmed URL, confidence and supporting evidence as configurable research outputs when review is useful; a different company can be a past role or side project, so investigate rather than treating it alone as a contradiction. Select the model from the shared AI model selector and set research limits separately. A cached replay is a reused result, not a new verification. Stop and explain a research failure using the returned error and stop reason instead of treating an empty output as not found.

- Filters are exact when the source owns the fact, approximate when they rest on labels or keywords. Verify approximate ones per row; never re-check exact ones.
- A Required role from a search index can be stale: confirm it before a paid step. A dated requirement needs a dated field.
- Judge by meaning: a plain equivalent of a role passes. Fail a row only on evidence that contradicts a criterion.

## Gates

- A paid run's ceiling is the preview maximum from `tables.table_runs.preview_cost` (`estimated_credits.maximum`) or from `tables.columns.preflight`. Run it under the paid-run consent rule. A hand-picked lower ceiling can skip pricier waterfall lanes and still bill the cheaper ones. `estimated_credits.minimum` is the expected band, not a ceiling you may shrink to.
- An unknown estimate is a one-row pilot, never a free run. Stop the batch when the pilot's yield is below the coverage that quote priced. Do not retry-patch misses: a completed empty cell is an answer. Read `tables.table_runs.effect_unknown` before any retry. Those receipts are not safe to retry automatically.
- `tables.columns.run` returns when the run is queued. Queued is not spent. `tables.table_runs.cancel` needs the run's current `state_version` and a reason. Status `cancelled` stopped immediately. Status `cancellation_pending` is recorded and the worker has not observed it yet. Work already dispatched is not recalled, and credits already spent are not refunded.
- Compare balance with `workspace.usage.status_get` against unit cost times scope before a live batch. If the balance cannot cover `estimated_credits.maximum`, stop and say so.
- A refusal names what to fix: fix that input and retry, three times at most, then report it. A refusal is an answer. Do not silently substitute another source or provider.
- A sourcing approval covers the pull only. Enrichment, CRM writes, and outreach on the landed rows are separate approvals. A new schedule (`tables.table_sources.schedule_set`) or a higher standing ceiling is a second approval. Changing cadence asks again.
- A missing account blocks running, never building: build everything and say what to connect.
- Never invent a count. Quote a match count only when `tables.sourcing.preview` returns one, and name that preview. When the preview says free preview is unavailable, report that and the smallest paid sample cost it returned. A target or a sample length is not a market size.
- Never delete rows unless the user asks. Removal is below.
- Keep email, LinkedIn URL and company domain columns on people rows so do-not-contact matching works.

## Remove rows

Read the matching rows first with `tables.rows.query` (same worksheet, same filter, the columns you will judge). Delete that same set with `tables.rows.bulk_delete`: pass those `row_ids`, or one `selection_snapshot_id` from `tables.selection_snapshots.create`, and `confirmed: true`. An unqualified delete of every row is not this step. Read the survivors with `tables.rows.query`. The receipt is the check. `tables.rows.restore` clears `deleted_at` on one row and does not recover a cell that was removed separately. Removed values stay readable with `tables.rows.history`. Deleting rows does not stop sequence enrollment.

## If the source or the column fails

The attach, preview, dry run, and live params are in [sourcing.md](sourcing.md). Which column kind answers the question is in [column-authoring.md](column-authoring.md). What to cite after the run is in [readback.md](readback.md). Waterfall enrichment of rows you already have is [`enrichment`], after Required criteria have passed.

| What came back | What to do |
| --- | --- |
| `tables.sourcing.preview` returns no rows | Loosen one filter. Preview again. Do not attach. |
| The preview says free preview is unavailable | Report that and the smallest paid sample cost it returned. Do not invent a count. |
| Warning `qualification_columns_missing` | Stop. Configure the chain. Do not spend. |
| The source drops a filter | Report that filter by name. Keep the criterion as a run condition. Do not say the source applied it. |
| `tables.table_runs.wait` returns `timed_out` true and `terminal` false | The run is still open. A count is unknown, not zero. Wait again. |
| `tables.table_runs.get` shows `checkpoint.resumable` true | `tables.table_runs.resume` continues the remaining work and does not redo completed units. It needs the current `state_version`, the checkpoint token, and an idempotency key. It spends again for the remainder, under the paid-run consent rule. |
| The run failed and it has no checkpoint | `tables.table_runs.failure_report` is the per-row detail (`error_category`, `retryable`, `why`). `tables.table_runs.retry` starts a fresh run of the same binding, spends again, and reuses nothing. Its returned `run_id` is the original run id, so poll `tables.table_runs.get`. Do not retry when a checkpoint exists. Use resume. |
| `error_category` `configuration` or `credentials` | Fix the named input or connect the account the run named. A missing account blocks running, not building. |
| `error_category` `budget` | Read `workspace.usage.status_get`. If `credits_remaining` is below `estimated_credits.maximum`, do not start another batch. |
| `tables.table_runs.effect_unknown` lists the run | Do not retry and do not resume. Those receipts are still calling or have an unknown outcome. |
| A completed cell is empty | That miss is the answer. Do not patch it from a similar row. |
| Some rows succeeded, some failed, some are empty | Read `tables.table_runs.get`, `tables.table_runs.failure_report`, and `tables.cells.read_page`. Report the four outcomes separately. Do not call the run complete. |
| A refusal names an input | Fix that input and retry, three times at most. Do not swap the source or the provider. |

## Done

Report the workbook link, sample size, qualified count against the target, qualification rate, credits charged, accounts to connect, and next steps. If this document disagrees with a tool result, the tool is right.

## X enrichment columns

Use X profile or X recent posts with an X handle, profile URL, or user ID. X profile returns profile details and counts. X recent posts returns the latest posts and their count. X charges per profile and per post returned.
