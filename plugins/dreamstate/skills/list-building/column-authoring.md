# Designing columns

A column answers one question about a row: what value it holds, what produces it, and which rows are worth spending on. Decide all three when you add it.

## Add a column

- Find enrichments with the findable catalog for this worksheet; choose by the data you need, never by provider. Use the returned add call unchanged: its input bindings name existing column keys only.
- Reference source fields and columns exactly as the worksheet returns them after the source lands: `{{column_key}}` or `{{column_key.field}}`. Never guess a field name or add a copy column to reach one.
- Send a declared output only for definitions that let you declare it; fixed-output enrichments return their own fields.
- Save the run condition with the column (see the skill). Read the column back and confirm it.
- Adding a column costs nothing. Run it on one row with a credit ceiling, read the cell, then run the rest with `max_rows` set, continuing while more rows remain.
- A refused column names the input to fix: fix it rather than switching to another kind of column.

## Criterion columns

- When a keyword or list match on an existing value decides the criterion, it is a run condition, not a column. Use a formula only to show a value; an AI or research column only when the criterion needs judgment or outside evidence, and its result carries a one-line reason.
- Prompts reference row columns and the user's criteria, never sample companies, people or values.
- The fit score is where criteria meet: its rubric names each criterion as Required or Nice to have and uses the row's signals as evidence. Contact data never proves anything else.
- A blueprint's AI fit score arrives with an empty instruction: it holds the row fields as context but no rubric, so it does not run until one is written. Before running it, read the workspace Company Context and ideal customer, write a short rubric naming who scores high and who scores low for this offer, end with "Return only an integer from 0 to 100.", and keep the row field lines below it. If there is no Company Context yet, ask the user who they sell to rather than inventing a generic profile.

## Formulas

Read the formula definition's `expression` description first: it lists every helper, operator and limit. Regex and URL parsers do not exist.

```text
UPPER({{company_name}})
{{company_profile.employee_count}} >= 50 ? 'target' : 'small'
LAST({{company_linkedin_url}}, '/')
```

## Changing a column

Read its current state before changing it, so an edit never overwrites a newer change.

## Clay CSV rebuilds

CSV values are baseline evidence, not recovered Clay configuration. Ask once for missing Clay prompt, formula, waterfall, and HTTP/tool settings. If user lacks them, read headers and values, then supply only catalog-backed inferred equivalents: enrichment, AI/research prompt marked `# approximated, original Claygent prompt not recovered`, formula reference, provider waterfall in supported catalog order, or native tool mapping. Mark inferred plans approximate. Create columns disabled, preserve raw Clay column, request `tables.table_runs.preview_cost` for each rebuilt column. Never invent provider names, tool ids, or capability ids. No catalog equivalent means keep raw column and report reason.
