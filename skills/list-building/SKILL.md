---
name: list-building
description: "Build a workbook of companies or people: as few sources as possible, people at companies through the employees enrichment fanned out to a second worksheet, one column per criterion marked Required or Nice to have, run conditions that skip rows already failing, an optional score, sample then scale, and honest coverage."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# Workbook building

## Boundary

Owns: sources, columns, criteria, and building qualified lists.
- Multi-step outreach sequences: [`outreach`].
- Existing worksheet structure and values: [`worksheet-tidy`].
- Durable customer and company records: [`crm`].
- Warm sources: [`linkedin-warm-signals`].
- Explaining cell values and failed runs: [`diagnostics`].
The `tables` pack lists the tools and their exact inputs. When user says delete, permanently delete named workbook, worksheet, template, automation, contact, row, or column after showing removal impact and receiving explicit approval. When user says archive, archive only. Never say permanent deletion is unavailable.

A list is worth contacting when every row would recognise itself in the reason you contact it. Find the smallest set where that is true, prove it on each row, and stop.

## Rows and qualified rows

- One workbook per list; each worksheet holds one kind of row and at most one live source. Everything else is a column.
- A row is a raw entry from the source. A qualified row passes every Required criterion.
- Track every criterion the user named, even when the source already filtered on it, and mark it Required or Nice to have. A criterion is a condition, not a column: put it at the cheapest place that decides it (a source filter, a run condition on a value already on the row, or a condition in the fit score). Its result is pass, fail or unsure. Missing data is unsure, never a fail.
- Keep disqualified rows: they are paid for. Views decide what to show.
- Exclude competitors by default: rows at a competitor company (named in request or Company Context) and competitor companies themselves fail a Required criterion, applied in first paid column run condition. Say that you excluded them and which. When exclusion is required and Company Context plus request contain no competitor values, ask once for competitor names before any paid sample. Do not source approximate list, create paid rows, or substitute keyword exclusion.

## Blueprint or build

Before building, see if a blueprint does exactly what you need: its source is the right source for the hardest criterion and its columns are the chain you would build. If one does, install it, then configure its columns. If not, build your own workbook. Never install a near miss to have something to start from.

When asking for a blueprint recommendation, pass `required_filters` for every user-stated hard constraint, using its closed filter enum. Read `unmet_required_filters`; do not install a recommendation that cannot apply every Required filter.

For people engaging with competitor company posts, `signal-engaged-company-posts` exact match. Prefer it. It sets `max_posts_per_company=2`, `max_engagers_per_post=25`, `sample_size=25`, then excludes employees of selected companies in first paid qualification. Attach the company engager source directly only when a custom workbook is needed: send `company_urls` as literal string array, keep same caps, add employee-exclusion qualification before outreach.

## Free preview before build, complete chain before paid sample

Use the zero-credit source preview before workbook creation when the source permits a provider preview. That preview is a free read: `live` or `smoke_test`, not `dry_run`. Pass `definition_id`, typed `filters`, and `limit` directly. It returns named sample rows, writes no worksheet, and charges zero credits. Do not attach source or run it live merely to see first LinkedIn people rows.

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
- Read the column back after saving and confirm the condition is there.

Disqualifiers and the fit score:
- A disqualifier the row already decides is a run condition, never a new column. Role or headline keywords: one `not_contains` condition per word (student, intern, university) on the title or headline. Competitors: company name `not_in` the competitor names from Company Context. Size and dates: numeric ranges. Put them on the first paid column and on the enrollment gate.
- To add or change exclusions on an existing worksheet, edit the run conditions of its existing columns. Do not add columns for them.
- The AI fit score is one score column whose rubric lists every criterion as Required or Nice to have, with the row's signals (engagement, hiring, recency, role) as evidence. A Required fail scores 0; missing data is unsure. Enroll only rows at or above the threshold the user set.
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

- Filters are exact when the source owns the fact, approximate when they rest on labels or keywords. Verify approximate ones per row; never re-check exact ones.
- A Required role from a search index can be stale: confirm it before a paid step. A dated requirement needs a dated field.
- Judge by meaning: a plain equivalent of a role passes. Fail a row only on evidence that contradicts a criterion.

## Gates

- A paid run's ceiling is its preview's maximum. Run it under the paid-run consent rule.
- A refusal names what to fix: fix that input and retry, three times at most, then report it.
- A missing account blocks running, never building: build everything and say what to connect.
- Never invent a count. Never delete rows unless the user asks.
- Keep email, LinkedIn URL and company domain columns on people rows so do-not-contact matching works.

## Done

Report the workbook link, sample size, qualified count against the target, qualification rate, credits charged, accounts to connect, and next steps. If this document disagrees with a tool result, the tool is right.
