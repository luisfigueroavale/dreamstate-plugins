---
name: visibility
description: "Get found by search engines and answer engines. Pick the query set, gather crawl, index, mention and citation evidence, fix pages, then measure the change."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Visibility

## Boundary

- Owns: choosing queries and measuring search and answer-engine visibility.
- Drafting and publishing articles: [`writing`].
- The `visibility` pack lists the tools and their exact inputs.

Capturing LinkedIn engagers and other warm signals belongs to [`linkedin-warm-signals`].

Improve discoverability and qualified organic traffic in traditional search and in generative answer engines, as one practice. This skill owns search evidence, crawlability, page structure, technical readiness, tracked prompts and citation measurement. It hands evidence-backed page briefs to `writing`, which authors the content.

## Operating contract

```operating-contract
actions: []
```

Read measured evidence, propose prioritized changes, and treat paid or asynchronous measurement as bounded work: show scope and cost, then run it under the paid-run consent rule. Before a sensitive change, show the evidence and the exact change. Knowledge owns business facts; writing owns content changes.

## Loop

1. Resolve the exact site, then read brand, audience, conversion goal and competitors.
2. Check the search connection before using its data, and keep property, date range, freshness and completeness with every number. See search-evidence.md.
3. For a named page, inspect that page now; never assume an earlier audit still describes it. See technical.md.
4. Before adding prompts, read the tracked query set and its status. See query-sets.md.
5. Start a new probe only when the user wants new engine observations; a metric refresh recomputes existing ones. See probes.md.
6. Prioritize by business impact, effort, confidence and dependency, and state every assumption.

## Evidence rules

- An observation is what a capability returned for a stated window; everything else is inference. Say which is which.
- Movement claims compare a window with the preceding equal-length window. Missing rows do not prove rank zero.
- Each technical check answers its own question; one passing check is never an overall health claim.
- A scan or refresh may spend credits or run asynchronously. Wait for its terminal state: a queued scan is not a result. Refresh only when existing evidence is too stale for the decision.
- Crawler fetches, AI referral traffic and probe recommendations are separate evidence; never fold them into one claim. Absent crawler evidence is not proof of blocking.
- A generated proposal for a site file is a reviewable draft; generating it neither installs nor proves it was read.

## Exact denominators

Prompt visibility is citations or qualifying mentions divided by completed runs for that exact prompt, engine set, locale and window. Report run count, citation count and percentage together. Zero citations over zero runs is unavailable, not 0%. Weight averages by their denominators and never merge engines, locales or windows silently. Sentiment uses its own total. Citation share, mention rate and referral traffic are distinct and never one score.

The tracked query set is a revisioned instrument: changing a prompt's wording breaks the comparison, so change it deliberately and disclose the break.

## GEO answer worksheet

For direct answer-engine evidence in a workbook, discover the current worksheet source and its exact schema before binding prompts, brand details and competitors. Read existing rows and their freshness before commissioning another run.

## Handoff to writing

A brief names the target query and intent, the evidence window and property, current and previous performance, the destination page or a new page, observed competing results, the conversion goal, internal links, and evidence gaps. When an existing page owns the intent, request a revision instead of a duplicate.

## Pick the path

| Question | Call | Not this |
| --- | --- | --- |
| What is already measured | `visibility.overview`, `visibility.citations`, `visibility.analytics_query`, `visibility.report_get` | `visibility.probe.start` |
| New answer-engine observations | `visibility.tracked_prompts.status`, then `visibility.probe.start` under the paid-run consent rule, then `visibility.probe.status` until terminal, then `visibility.probe.results` | `visibility.refresh`. That call re-collects citation data on a different pipeline. |
| Crawl, robots, speed, or page HTML | `visibility.workspace_site_get`, `visibility.sitemap_get`, `visibility.seo.robots_audit`, `visibility.pagespeed_get`, `visibility.html_analysis_get` | A probe, and a claim that one passing check is site health |
| Change the query set | `visibility.tracked_prompts.generate` does not save. `visibility.tracked_prompts.create` adds one. `visibility.tracked_prompts.save` replaces the set. | Saving a generated list with `save` when you only meant to add one |
| A stored report | `visibility.report_generate` writes a new row. `visibility.report_get` reads it. `visibility.reports_list` is metadata only. | Treating the generate receipt as the rendered report |

Layer detail is in [probes.md](probes.md), [query-sets.md](query-sets.md), and [technical.md](technical.md).

## Worked report and recovery

For an existing probe, read visibility.workspace_site_get with {} and visibility.tracked_prompts.status with {}. Read visibility.probe.status and visibility.probe.results with {batch_id: returned batch ID}. Once terminal, report the site, prompt revision, engine, locale, window and completed versus expected runs. For example, three citations from ten completed comparable runs is 30%; zero completed runs is unavailable. Name the weakest prompt and give writing an evidence-backed page brief.

If the batch is running, wait and reread instead of presenting the start receipt as a finding. If it fails or is partial, inspect the error and completed evidence, report coverage, and correct the identified cause before considering another run. Prefer free reads, then a dry run or preview when available, followed by a small pilot with an explicit ceiling and approval under the paid-run consent rule. Never infer a missing result is zero.

## Second report: crawl, not a new probe

The person asked what blocks discovery. Do not start a probe.

`visibility.workspace_site_get` with `{}`. If `site` is null, `visibility.workspace_site_ensure` is the create, and there is nothing to audit until a domain exists. With a domain, `visibility.sitemap_get` with `{}`. Trust `sitemap.ok`. `sitemap.ok` false with `fetch_failed` or `parse_failed` is a failed fetch, not an empty site. `visibility.seo.robots_audit` fetches robots.txt and returns `bots[]` with `status` `allowed`, `blocked`, `not_applicable`, or `error`. It never edits the file. `visibility.pagespeed_get` with `{}` is the cached audit. `is_stale` true means a side is missing or older than 25 hours. Refresh with `visibility.site_scan` only when that cache cannot support the decision. `visibility.html_analysis_get` is the cached homepage parse. It is null when no brand site is configured.

`visibility.citations` with `{}` is the stored citation merge. It does not collect. `visibility.report_generate` stores a new growth report and does not email it. `visibility.report_get` with the returned report id is the readback. `payload` is the report. `visibility.reports_list` does not include that payload. Say which checks ran, which were stale, and that no probe was started.

## Done

One report covers search and answer engines: site and property, window, freshness and completeness, measured findings, assumptions, prioritized actions, query set revision, engines covered, completed versus expected runs, denominators, and the weakest prompts by name. Never claim an improvement until a later equal-scope measurement shows it. When blocked, say what is missing, what was already measured, and the one read or decision that unblocks. Read back what you changed before reporting.
