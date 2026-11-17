# Sourcing rows

## Attaching a source

- A worksheet has one source. Attaching it is building: it saves even when an account is still missing; the run says what to connect.
- Bind the source's declared inputs with values from the request, in the form each input declares. Never an empty binding.
- When the user gives a LinkedIn people-search URL, attach `provider.linkedin.people.search_by_url` with that exact URL as `raw_search_url`. It runs the search the user built and saw. Never take the URL apart into filters for `provider.linkedin.people.search`: ids lose their names and the rows drift from what the user saw.
- LinkedIn keyword search applies AND before OR. Group every OR inside parentheses (`("founder" OR "co-founder") AND "she/her"`). When a URL's keywords mix AND and OR without parentheses, say how LinkedIn will read them before the sample spends credits.
- Prefer source whose declared filters apply every Required criterion before provider spend. Resolve candidate source definitions through `tables.executable_definitions.list_resolved`; select ready hard-firmographic people source declaring `company_size` and `industry`. Connected `provider.linkedin.people.search` does not declare company headcount. For required headcount plus industry, use resolved hard-firmographic people source, or company-first `source.company_search` then qualified-company people fan-out. Keywords such as `SaaS` never satisfy company-industry or headcount criteria.
- Attach dry run returns `binding_preview.binding`: exact normalized binding live accepts. Use returned refs unchanged. Human LinkedIn location labels resolve during attach dry run to `{id,label}` refs; do not perform separate taxonomy lookup when dry run supplies resolved ids.
- Do not pass `columns` on attach unless a field should be its own column. Attach creates only the source column. `available_output_paths` lists every field. `field_guidance` names a few relevant `{{source_column_key.field}}` references. Add a formula column later only for a field you choose.
- To continue a list, run the same source again. Changing its inputs starts it over.
- Enrich rows you already have; never add a second source for them. Fan out only when an enrichment returns a nested list per row and the user wants those items as rows.

## Changing a source

- Read `tables.table_sources.list` first. Use the returned source id, state, binding revision and binding as the current state; never reconstruct a compiled binding or its digest.
- Author a source change with `tables.executables.save`: set target kind to `source`, copy the returned source id, state and binding revision into its target fence, and express only the intended change in `binding_spec`. The save path compiles the binding and preserves omitted existing source policy.
- Preserve an existing event trigger and its other policy unless the user explicitly wants to change it. A source can have valid demand policy before its provider account is connected; save that draft, connect when the run says it is needed.
- An explicit `tables.table_sources.run` does not require a policy edit. Run the current source when the user wants rows now; change its policy only when the future behavior should change.

## Social account posts and AI answers

- For competitor or creator content analysis, use `source.account_posts_<platform>` with pasted account links. On an existing account URL column, use `column.account_posts_<platform>`, then Fan out for one row per post.
- Attaching an account posts source lays out Post URL, Post text, posted date, media type and engagement columns automatically. Do not add formula columns to copy those fields.
- Reruns only buy posts not already held. These worksheets contain `content` rows, never an audience.
- For AI visibility, use `source.geo_ai_answers` with prompts and brand details. It creates one row per prompt, engine and day. Rerunning same day is free.

## Filter values

- Use the real options a filter offers for places, industries, companies, titles and schools. A guessed value returns wrong rows that look right: read the rows.
- A filter the source drops is not applied. Report each dropped filter by name. Keep that criterion as a column with a run condition. A live cascade never advances to a provider that drops an account scope the user required.
- Several roles are several title values, never one joined string.
- Hiring signals: the role being hired filters the companies; the titles of people to contact go into the employee lookup. The place goes in job location, not headquarters, unless the user asked where companies are based.
- Hiring at companies of a size: when the size band is the narrow part and the role is common, source companies with the size filter, add the open jobs column on each company, and run the employee lookup only on companies with at least one open job. When the role is rare, start from the hiring source and put size in the run condition of the first paid column. Say which path you chose and why.
- Company size, seniority and employment type: use the source's input when it has one; otherwise add the column and a condition, and say which.
- Recruiting and staffing firms post for clients. When the sample shows them, add one judgment column for the actual employer and whether the poster is a recruiter.

## Targets and credits

- Use the request's target. When none is given, free-preview first, then run the attached source with `max_rows` in the params of `tables.table_sources.run` (not on `tools_run`) and matching `max_credits`. When no target is given, start at `max_rows:25` and ask how far to grow after reading those 25 rows.
- Over-provision about 1.4 times the target when later criteria drop rows.
- `tables.sourcing.preview` is a free read. Door modes `dry_run` and `live` both execute this read. Pass definition_id and its source binding values inside source_inputs; keep execution mode at the door. Resolve the selected source input contract, including schema references. Flattened source fields and params.dry_run are rejected. For LinkedIn people search, it returns up to ten named provider rows free, with no workbook, source attachment, or credits. Use it before a paid sample. Preview maximum is ceiling; what was charged is read from the finished run. Counts before the run finishes are unknown, not zero.
- Run an enrichment on a few rows, read them, then continue the remaining rows instead of rerunning the sample.
- A cell blocked by a daily limit is the workspace's pacing cap, not a defect.

## Companies and people

Use the size field the resolved definition declares. A live employee count and a declared employee range are different inputs. A range can lag. Do not treat a range label as a live count, and do not invent `employee_count` on a definition that does not list it.

People at companies are priced per company and per page through `tables.sourcing.preview` or `tables.table_runs.preview_cost` before the fan-out. When the quote is unknown, buy one company first, read the people rows, then continue. The account scope survives a fallback. Continuing a search is another run of the same source, not a new source. Enrichment of those people is a separate approval.

## Imports

`tables.table_sources.csv_import_prepare` stages a file. Verify the staged row count against the source before any paid column runs. A blank on upsert clears the value. CSV rows are staging evidence, not recovered configuration. See column-authoring for a Clay CSV rebuild.

## Worked source run

The user wants 25 US heads of sales and asks to preview before spending.

`tables.executable_definitions.list_resolved` first. Copy the ready people source `definition_id` whose declared inputs cover title and country. Do not invent one.

`tables.sourcing.preview` is a free read. Door modes `dry_run` and `live` both execute this read and charge zero. Params:

```json
{
  "definition_id": "<id from list_resolved>",
  "source_inputs": {
    "<filter key from tables.sourcing.filters>": "<closed value that call listed>"
  },
  "limit": 10
}
```

`tools_get` `tables.sourcing.preview` and use the input names it declares. `tables.sourcing.filters` lists the closed values. Do not guess a label. Flattened source fields and `params.dry_run` are rejected. Read the named rows. If the preview says free preview is unavailable, stop and report the smallest paid sample cost.

`tables.table_sources.attach` only after the preview is acceptable. Dry run first. The receipt field `binding_preview.binding` is the binding the live attach accepts. Use those refs unchanged.

`tables.table_sources.run` dry run next. The quote is the credit estimate. The ceiling is that maximum, under the paid-run consent rule. The user named 25, so the sample size on this call is 25. Live uses the dry run `quote_hash`, `max_credits` from that quote, a fresh `idempotency_key`, and `approved` when the paid-run consent rule requires it.

`tables.table_runs.wait` until `terminal` is true. `timed_out` true with `terminal` false means the run was still open. Read cells with `tables.cells.read_page`. Counts before the run finishes are unknown, not zero. A refusal is reported as returned. Do not swap in another provider. Enrichment of the landed rows is not included in this approval.

## X posts flow

The catalog has no separate X-posts capability. The source is a definition id from `tables.executable_definitions.list_resolved`. Use `source.x_posts_search` for a topic or complaint, `source.x_profile_posts` for supplied accounts, `source.x_posts_by_url` for supplied posts, and `source.x_post_engagers` for people on those posts. Copy the id only when that call shows it `ready`. A blocker means do not preview it as if it will run. An app-only search credential does not imply likes or other engager endpoints.

`visibility.x.posts_list` and `visibility.x.posts_capture` read posts already stored in the workspace. They do not search live X and they do not land worksheet rows. Do not use them as the source.

Preview, still a free read:

```json
{
  "definition_id": "source.x_posts_search",
  "source_inputs": {
    "query": "cold email deliverability",
    "archive_mode": "recent",
    "sort_order": "relevancy",
    "max_items": 10
  },
  "limit": 10
}
```

`query` is required. `archive_mode` is `recent` or `full_archive` (default `full_archive`). `sort_order` is `recency` or `relevancy` (default `recency`). `max_items` is 1 to 100. Optional `start_time` and `end_time` are ISO timestamps with an offset, and the start must precede the end. Pass `account_id` only when the definition's blockers say a connected X account is required.

Read the named posts against the complaint rubric before attach. Keep one row per post. Do not collapse a person's posts before the qualification column has judged them.

Then the same attach dry run, column chain, and `tables.table_sources.run` as the worked source run above. The qualification column is `column.web_agent` added with `tables.columns.add` before the paid sample, using the prompt in [x-identity-research.md](x-identity-research.md). Gate that column on the complaint result. A missing LinkedIn match is unsure, not a fail. Contact enrichment runs only when the user asked and the verified profile URL is set.

Readback is `tables.table_runs.wait`, then `tables.cells.read_page`. Report source rows, qualified authors, authors researched, candidate profiles, and confirmed matches as separate counts, with the credits for each stage. A candidate URL in a summary is not a confirmed match.
