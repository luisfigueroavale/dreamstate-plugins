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

## Social account posts and AI answers

- For competitor or creator content analysis, use `source.account_posts_<platform>` with pasted account links. On an existing account URL column, use `column.account_posts_<platform>`, then Fan out for one row per post.
- Attaching an account posts source lays out Post URL, Post text, posted date, media type and engagement columns automatically. Do not add formula columns to copy those fields.
- Reruns only buy posts not already held. These worksheets contain `content` rows, never an audience.
- For AI visibility, use `source.geo_ai_answers` with prompts and brand details. It creates one row per prompt, engine and day. Rerunning same day is free.

## Filter values

- Use the real options a filter offers for places, industries, companies, titles and schools. A guessed value returns wrong rows that look right: read the rows.
- A filter the source drops is not applied: keep that criterion as a column with a run condition.
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
