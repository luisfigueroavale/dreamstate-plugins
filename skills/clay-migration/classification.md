# Column classification

`tables.clay_migrations.preview` classifies each header. `clay_table_json.columns[]` may include `header`, `clay_type`, `formula`, `prompt`, `providers`, `dependencies`, `method`, and `url_template`. Without that object, only the header text is available, and executable fields stay empty.

| What you see | `kind` | `proposed_definition_id` | Confidence | Question when metadata is missing |
| --- | --- | --- | --- | --- |
| `clay_type` `formula`, or header starts with `formula:` | `formula` | `column.formula` | `exact` if the expression validates, else `manual` | Formula source missing. Cell values are not the formula. |
| `clay_type` `claygent_research`, or header matches research or claygent | `research` | `column.research` | `exact` when `prompt` is present | Prompt missing. Cell text is not the prompt. |
| Header matches claygent or a standalone AI token, and it is not research | `ai` | `column.ai` | `exact` when `prompt` is present | Prompt missing. |
| `clay_type` `waterfall`, or header matches waterfall or work email | `provider_waterfall` | `column.provider_waterfall` | `approximate` | Provider order missing, or no name, domain, or profile column to bind. |
| `clay_type` `http_api`, or header matches http, api, or `tool:` | `http_request` | `column.http_request` | `exact` only for GET with method and URL | Method and URL missing. Non-GET is blocked pending explicit approval. |
| Header matches find people | `manual` | null | `manual` | Rebuild the query by hand. |
| Anything else | `manual` | null | `manual` | None, unless you expected logic. Keep the raw column. |

Supported formula functions after translation: `if`, `lower`, `upper`, `trim`, `len`, `left`, `right`, `replace`, `number`, `contains`, `includes`, `startsWith`, `endsWith`, `coalesce`, `ifEmpty`, `concat`, `round`, `Math`. Clay `{Header}` becomes `{{column_key}}`. `AND`, `OR`, `NOT`, `IF`, and `<>` are rewritten. Any other function name is listed in `manual_questions` and is not applied as if it worked.

Waterfall bindings use definition inputs the catalog can run (`full_name`, `company_domain`, `profile_url` when those columns exist). Clay's provider names are stored as a note, not as the live provider order. Tell the user the order was replaced. Hand the live waterfall to [`enrichment`].

HTTP GET can be proposed only from metadata. A POST, PUT, PATCH, or DELETE stays a question. Do not downgrade it to GET.

`confidence` `approximate` writes provenance "approximated from Clay header; original settings not in export". Do not describe that column as a recovered original.

Each plan entry has `source_header`, `raw_column_key`, `kind`, `proposed_definition_id`, `proposed_binding`, `dependencies`, `confidence`, `manual_questions`, and `rebuilt_column_key` (`raw_column_key` plus `_rebuilt`). Apply skips entries with no definition or no binding, including every manual column. If nothing writable remains, apply returns 400 `clay_migration_configuration_required` and `remaining_manual_questions`.
