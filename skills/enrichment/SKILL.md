---
name: enrichment
description: "Enrich worksheet rows and single records: work and personal email, phone, company and person profiles, LinkedIn URL and verification, through waterfall columns with a free preview, a one-row pilot and a bounded batch under a credit ceiling. A miss is an answer; phone and personal email need their own yes."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Enrichment

## Boundary

- Owns: waterfall columns on worksheet rows, and enrichment of one CRM record.
- Qualified lists and the rows those columns run on: [`list-building`].
- Rebuilding a Clay table's enrichment columns: [`clay-migration`].
- The `tables` pack lists the tools and their exact inputs.

Use this skill when the user wants a work email, a personal email, a phone, a LinkedIn URL, company firmographics, or a verification verdict on rows they already have, or on one record. Do not use it to find the people (list-building), to enroll or send (outreach), or to merge CRM records (crm). A found value is not permission to email, call, or enroll. Those are separate approvals.

One-record calls in the crm pack are named below. Resolve their inputs with `tools_get` before the run.

## Start from the catalog

Free reads, in this order, before any column is added:

1. `tables.enrichment.catalog` lists what a worksheet can fill (person profile, work email, phone, headcount, funding, tech stack, hiring, news, signals). Each intent has status `default`, `selectable`, `no_default`, `coming_soon`, or `rejected`. A `no_default`, `coming_soon`, or `rejected` intent is a refusal. Report it and do not build a column for it.
2. `tables.enrichment.presets_list` returns named recipes and ready-to-add column templates. Definitions stay authoritative. A template carries the exact `tables.columns.add` payload, inputs, row gate, outputs, and credit estimate. Copy that payload. Do not hand-write a provider name.
3. `tables.executable_definitions.list_resolved` is the check when you need the live definition id and its input contract. Use the returned id. The catalog snapshot does not publish those definition ids as capabilities, so never invent one.

## Pick the path

| Data | Start with | Do not |
| --- | --- | --- |
| Work email on worksheet rows | `tables.enrichment.catalog`, then the work-email template from `tables.enrichment.presets_list`, copied into `tables.columns.add` | Run it on rows that already failed a Required criterion |
| Work email on one CRM record | `crm.records.tool_run` with the work-email definition from `tables.executable_definitions.list_resolved` | Pass a phone `provider_order` |
| Work email for one person, no worksheet and no record | `crm.contact.find_email` dry run, then live. Inputs are a LinkedIn URL, or a name plus a domain | Create a column for a single ask |
| Phone | Its own yes. The phone template from `tables.enrichment.presets_list`, or `crm.contact.find_phone` for one person | Treat a work-email yes as cover. `crm.contact.find_phone` spends 120 credits per call |
| Personal email | Its own yes, and only for ad-audience uploads, recruiting, or founder outreach | Add it because work email was approved |
| LinkedIn URL | The catalog intent when it is `default` or `selectable`, or `crm.contact.find_linkedin` for one person (2 credits per call, a miss costs 0) | Invent a URL when the lookup misses |
| Company firmographics (headcount, funding, tech, hiring) | `tables.company_enrichment.preview` for up to 50 companies, then `tables.table_sources.attach` and `tables.table_sources.run` | Call `tables.company_enrichment.run` live. It refuses with `not_executable` |
| Verification of addresses you already have | `tables.contacts.verify_email` | Use it to find addresses. It only grades the ones you pass |
| One attribute queued on one record, no definition binding | `crm.record_enrichment.create`, then `crm.record_enrichment.get`, then `crm.records.get` | Use it when `crm.records.tool_run` can take the binding. There is no batch form |

`tables.enrichment.apply_default_cascade` writes the server default provider waterfall onto one manual or enrichment column. The order is the billing lanes of the chosen preset when those lanes are legal waterfall steps. It is idempotent when the column already has that order. It calls no provider.

Enrich only rows that already survive Required criteria. `tables.rows.dedupe` finds duplicate pairs on a normalized email, company domain, or LinkedIn URL, or on a fuzzy person name plus company. The preview writes nothing. Approving its preview hash fills blank cells on the kept row and removes the duplicate only when both rows sit on this worksheet. Never pay twice for a duplicate or for a row you will discard.

## Safety ladder

1. Free preview. `tables.columns.preflight` quotes one enrichment column with no provider call and no charge. A BYOK lane reports `byok_platform_fee_credits_per_call`, including a successful miss. Vendor charges stay on the provider account. `tables.table_runs.preview_cost` prices a worksheet run and returns `estimated_credits.minimum`, `estimated_credits.maximum`, `preview_id`, and `expires_at`. Nothing is charged. `tables.company_enrichment.preview` inspects up to 50 companies (missing fields, routing, estimates). Unknown values are null. A company with no name, domain, or LinkedIn URL is unpriced. `tables.worksheet_enrichment.preflight` counts company rows and returns provider order, per-row estimate, and total. `crm.contact.find_email` dry run returns the profile and chain for free.
2. One-row pilot. After the paid-run consent rule is satisfied, run one row. Read the cell with `tables.cells.read_page` and the credits on the run. An unknown estimate is a one-row pilot, never a free live run.
3. Bounded batch. `tables.columns.run` queues one column over a frozen row selection. Put `max_rows` in its params, not on `tools_run`. The credit ceiling is `estimated_credits.maximum` from the quote. A lower hand-picked ceiling can skip pricier lanes and still bill cheaper ones. Then `tables.table_runs.wait` and `tables.table_runs.provider_summary`.
4. Stop the batch when the pilot's yield is below the coverage that quote priced.

`tables.company_enrichment.run` dry run returns the request and estimate. Live mode refuses with `not_executable`. Build company rows with `tables.table_sources.attach` then `tables.table_sources.run`.

Own-key lanes are real spend plus the platform fee `tables.columns.preflight` reported. They still follow the paid-run consent rule.

Phone and personal email need their own explicit yes. `crm.contact.find_phone` spends 120 credits per call. Personal email is for ad-audience uploads, recruiting, or founder outreach only. A work-email approval does not cover either. `crm.contact.find_linkedin` spends 2 credits per call (a miss costs 0 on the current flat price). `crm.contact.find_person` spends 0.9 credits per call. Do not add those columns, and do not call those lookups, unless the user named that data.

## A miss is an answer

A completed run with an empty value found nothing. `crm.records.tool_run` skips an empty extracted value and leaves it out of `updated_attr_keys`. Do not rerun that definition on the same inputs. Do not add another provider unless the waterfall order already includes that lane and this run stopped before trying it. A `no_default` catalog row is not permission to pick a substitute.

`tables.table_runs.effect_unknown` lists provider receipts still calling or with an unknown outcome. Those calls are not safe to retry automatically. Read it before any retry.

Queued is not spent. `tables.columns.run` returns when the run is queued. `tables.table_runs.cancel` needs the current `state_version` and a reason. Work already dispatched is not recalled, and credits already spent are not refunded.

## Verification

`tables.contacts.verify_email` checks up to 100 supplied addresses. It does not find addresses or create contacts. Each verdict is `valid`, `invalid`, `catchall`, or `unknown`.

- `unknown` means absent, not bad. Keep the row. Do not drop it and do not treat it as invalid.
- `catchall` stays unconfirmed. Do not treat it as valid unless the waterfall's own later step escalates it, and that step was part of the approved order.
- `invalid` is the only verdict that means the address failed verification.

## One record

`crm.records.tool_run` runs one executable definition against one record and writes enrichment-class attributes. The per-call credit ceiling is 50, not a fixed charge. Dry run first. `binding.definition_id` must be an active non-source definition from `tables.executable_definitions.list_resolved`. Every `record_attribute` input binding must point at a populated attribute of the matching type, or the run is refused before any credit is spent. Copy the binding the definition list returned. Do not hand-write `input_binding_digest` or `schema_digest`. The server computes them when omitted. A supplied mismatch is refused.

`output_writes` is 1 to 32 of `{ attr_key, output_path }`. `attr_key` comes from `crm.record_objects.attributes_list` and must be writable. Only `output_writes[0]` steers the provider call. The rest are read from the same payload. `output_path` is a dotted path such as `email` and must match a declared output field.

`provider_order` is optional and steers which paid vendors run, and in what order. The catalog allows it only on the work-email definition and the phone definition. Any other definition returns `400` `record_tool_provider_invalid` and spends nothing.

Work email and phone do not share a list. Omit `provider_order` to use the workspace saved waterfall for that field, which falls back to the registry default, cheapest first.

- Work email, cheapest first: `blitz`, `findymail`, `prospeo`, `fullenrich`.
- Phone, cheapest first: `blitz_phone`, `findymail_phone`, `prospeo_mobile`, `fullenrich`.

`blitz` is not a phone id. `blitz_phone`, `findymail_phone`, and `prospeo_mobile` are not email ids. `fullenrich` is on both lists. Copying the email list onto a phone run, or the reverse, returns `400` `record_tool_provider_invalid` and spends nothing. Ids must be unique. Billing is flat per successful find, so order changes coverage and latency, not the price of a hit. A phone order is never permission to run phone. Phone still needs its own yes.

Read the record back with `crm.records.get`. `cost_credits` is what was charged. `provider_id` is the provenance. A miss stays a miss.

`crm.record_enrichment.create` is the other one-attribute door. It queues one attribute on one record, returns `202` with `{ job }` and no value, and is already committed to a provider. Poll `crm.record_enrichment.get` for job state, then `crm.records.get` for the attribute. There is no batch form. Use `crm.records.tool_run` when you have the definition binding. Use `create` only when the user asked to queue one attribute that way.

One-shot lookups, when there is no worksheet and the user asked for one person: `crm.contact.find_email` (LinkedIn URL, or name plus domain), `crm.contact.find_phone` (its own yes), `crm.contact.find_linkedin`, `crm.contact.find_person`. Dry run `find_email` before live. These do not create a worksheet column.

`crm.enrichment_preset.get` reads the workspace default CRM preset, or null. `crm.enrichment_preset.set` saves the attribute selection for future records and does not backfill and does not start a run.

## Worked example

The user has a people worksheet whose rows already pass the Required fit score, and asks for work email on those rows only.

1. `tables.rows.query` the qualified rows. `tables.rows.dedupe` if identity columns show duplicates.
2. `tables.enrichment.catalog`. Confirm work email is `default` or `selectable`. If it is `no_default`, stop and say so.
3. `tables.enrichment.presets_list`. Copy the work-email template's `tables.columns.add` payload, including its input bindings and row gate. The gate requires the LinkedIn URL or the name and domain the template names, with `is_set`.
4. `tables.columns.add`. Read the column back with `tables.columns.list`. Adding spends nothing and runs nothing.
5. `tables.columns.preflight` for that column, then `tables.table_runs.preview_cost` for one row. Note `estimated_credits.minimum` and `estimated_credits.maximum`.
6. One-row `tables.columns.run` under the paid-run consent rule. `max_rows` is 1 in params. `max_credits` is the quote maximum. Then `tables.table_runs.wait` and `tables.cells.read_page`.
7. If the cell is empty, stop. That miss is the answer. Do not run a second provider by hand.
8. If the cell is populated, run the remaining qualified rows with `max_rows` set to that set and `max_credits` set to the new quote's maximum. Read `tables.table_runs.provider_summary`.
9. Do not enroll the rows. Report how many rows were quoted, how many returned an address, how many missed, and `cost_credits`.

One record, after the user points at a person and asks for a work email.

`crm.records.get` for the person. `tables.executable_definitions.list_resolved` for the work-email binding. `crm.record_objects.attributes_list` for a writable email `attr_key`. Dry run `crm.records.tool_run` with no `provider_order` unless the user named vendors:

```json
{
  "record_id": "<from crm.records.get>",
  "binding": "<the binding object list_resolved returned>",
  "output_writes": [{ "attr_key": "<writable email attribute>", "output_path": "email" }]
}
```

The door is `mode` `dry_run`. That call writes nothing. Its `quote_hash` and credit quote are the price, not a confirmation.

Live uses the same params, the dry run `quote_hash`, `max_credits` at that quote and at most the capability ceiling of 50, a fresh `idempotency_key`, and `approved` when the paid-run consent rule requires it. A quote on a refused live call is only a price. Read `crm.records.get`. Report `cost_credits`, `provider_id`, and `updated_attr_keys`. An empty `updated_attr_keys` is a miss.

Verification, after the user hands you addresses and asks only to grade them. `tables.contacts.verify_email` takes `addresses`, 1 to 100, no duplicates. It does not find addresses. Dry run under the paid-run consent rule, then live with that `quote_hash`. Readback is one verdict per address: `valid`, `invalid`, `catchall`, or `unknown`. Keep `unknown`. Leave `catchall` unconfirmed. Only `invalid` means the address failed.

## If it goes wrong

| What came back | What to do |
| --- | --- |
| `not_executable` on `tables.company_enrichment.run` live | Stop. Use `tables.table_sources.attach` and `tables.table_sources.run`. |
| `400` `record_tool_provider_invalid` | Drop `provider_order` or keep only the work-email or phone definition that accepts it. Nothing was charged. |
| Input binding refused before spend | Fix the attribute or column the definition named. Do not switch column kind. |
| Completed cell empty | Miss. Do not rerun the same definition. |
| Verdict `unknown` | Absent, not bad. Leave the row. |
| Verdict `catchall` | Unconfirmed. Do not call it valid. |
| `tables.table_runs.effect_unknown` lists the run | Do not retry until a person has checked every destination. |
| Quote has no credit estimate | One-row pilot only. |
| Balance from `workspace.usage.status_get` is below `estimated_credits.maximum` | Do not start the batch. Say the gap. |
