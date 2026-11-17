# Designing columns

A column answers one question about a row: what value it holds, what produces it, and which rows are worth spending on. Decide all three when you add it.

## Which kind

| Need | Use | Do not use |
| --- | --- | --- |
| Show or reshape a value already on the row | A formula. It evaluates on read and spends nothing. | An AI column, which can invent a missing fact. |
| Judge text the row already holds | `column.ai` for an open answer, or `column.ai_fit_score` for a 0 to 100 fit score. An AI column has no web access. Asked to check a website, it answers from the row and can invent. | A research column, which is the paid web read. |
| A fact that lives on the web, or on a page the row links | `column.research`. Output is fixed `{status, answer, evidence}`. `found` false, or an empty answer, is unknown, not a contradiction. | An AI column. |
| Work email, phone, LinkedIn URL, company data, or verification | The enrichment skill. Resolve the definition with `tables.enrichment.catalog` and `tables.enrichment.presets_list`. | A prompt that asks a model to guess an email. |
| One provider operation the catalog already lists | The definition `tables.executable_definitions.list_resolved` returns. Copy its add payload. | A hand-written provider name. |

`tables.columns.add` writes the definition and spends nothing. It does not run the column. Price with `tables.columns.preflight` (one enrichment column, no provider call) or `tables.table_runs.preview_cost` (`estimated_credits.minimum` expected, `estimated_credits.maximum` worst case, plus `preview_id` and `expires_at`). The ceiling for the paid-run consent rule is that maximum. A BYOK lane on `tables.columns.preflight` reports `byok_platform_fee_credits_per_call`, including a successful miss. Vendor charges stay on the provider account and still need approval.

A non-GET HTTP column is an external write and needs its own approval. Mirror the provider id, status, error, and synced time onto the row. Read the column back before a second run so you do not repeat a call whose first effect is still unknown (`tables.table_runs.effect_unknown`).

## Add a column

- Find enrichments with the findable catalog for this worksheet; choose by the data you need, never by provider. Use the returned add call unchanged: its input bindings name existing column keys only.
- Reference source fields and columns exactly as the worksheet returns them after the source lands: `{{column_key}}` or `{{column_key.field}}`. Never guess a field name or add a copy column to reach one.
- Send a declared output only for definitions that let you declare it; fixed-output enrichments return their own fields.
- Save the run condition with the column (see the skill). Read the column back and confirm it.
- Adding a column costs nothing. Run it on one row with a credit ceiling, read the cell, then run the rest with `max_rows` in the params of `tables.columns.run`, not on `tools_run`, continuing while more rows remain.
- A refused column names the input to fix: fix it rather than switching to another kind of column.

## Criterion columns

- When a keyword or list match on an existing value decides the criterion, it is a run condition, not a column. Use a formula only to show a value; an AI or research column only when the criterion needs judgment or outside evidence, and its result carries a one-line reason.
- Prompts reference row columns and the user's criteria, never sample companies, people or values.
- The fit score is where criteria meet: its rubric names each criterion as Required or Nice to have and uses the row's signals as evidence. Contact data never proves anything else.
- The fit score column is `column.ai_fit_score` (TypeSafe, fixed numeric `score` from 0 to 100). column.ai_fit_score is the way to score fit, ICP, or lead qualification, and it is the cheaper option. Do not implement an overall fit, ICP, rank or rating score with `column.ai`. A narrow row classification such as first-person complaint versus promotion can use an ordinary AI boolean with its evidence-based explanation, then feed the fit score or a run condition. A first line, a summary, or any other open-ended judgment stays `column.ai`.
- A blueprint's AI fit score is already `column.ai_fit_score`. Its prompt starts with a default rubric, then the row fields. Before running it, read the workspace Company Context and ideal customer, replace that default rubric with a short rubric naming who scores high and who scores low for this offer, and keep the row field lines below it. Do not add a second generic AI column that returns an integer. Sorts and enrollment gates read the numeric `fit_score_value` formula (`fit_score.score`), not the `fit_score` object. If there is no Company Context yet, ask the user who they sell to rather than inventing a generic profile.

## Formulas

Read the formula definition's `expression` description first: it lists every helper, operator and limit. Regex and URL parsers do not exist.

```text
UPPER({{company_name}})
{{company_profile.employee_count}} >= 50 ? 'target' : 'small'
LAST({{company_linkedin_url}}, '/')
```

## Changing a column

Read its current state before changing it, so an edit never overwrites a newer change.

The same fence applies to the worksheet source: read `tables.table_sources.list`, then use `tables.executables.save` with a source target and the returned state and binding revision. Put the intended authored change in `binding_spec`; never construct a compiled binding or digest. Omitted policy remains intact, including event triggers. A valid demand policy may be saved before a provider account is connected, and a one-off source run never needs a policy change.

## Research columns

Create research with `binding_spec`, not `prompt`: `definition_id: "column.research"`, only `input_bindings.query`, and query `{ "kind": "template", "parts": [{ "kind": "literal", "value": "What recent news matters for " }, { "kind": "column", "column_key": "company" }, { "kind": "literal", "value": "?" }] }`. Research output is fixed `{status, answer, evidence}`. Do not declare `output`.

## Clay CSV rebuilds

CSV values are baseline evidence, not recovered Clay configuration. Ask once for missing Clay prompt, formula, waterfall, and HTTP/tool settings. If user lacks them, read headers and values, then supply only catalog-backed inferred equivalents: enrichment, AI/research prompt marked `# approximated, original Claygent prompt not recovered`, formula reference, provider waterfall in supported catalog order, or native tool mapping. Mark inferred plans approximate. Create columns disabled, preserve raw Clay column, request `tables.table_runs.preview_cost` for each rebuilt column. Never invent provider names, tool ids, or capability ids. No catalog equivalent means keep raw column and report reason.
