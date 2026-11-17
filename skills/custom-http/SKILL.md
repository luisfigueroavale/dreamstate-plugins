---
name: custom-http
description: "Register, preview, run and remove a custom HTTP integration so a worksheet column or workflow can call the user's own API, with auth, inputs and a one-row test before any batch."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Custom HTTP

## Boundary
- Owns: Discovery and credential handling for custom HTTP integrations.
- The `integrations` pack lists the tools and their exact inputs.

Use this when the user wants a credential on a workspace custom HTTP integration, or wants to know whether a custom API can be registered and run. Worksheet columns and enrichment belong to [`list-building`] and [`enrichment`]. A durable graph belongs to [`workflows`]. A one-off research fetch is not a reason to attach a credential.

## What the catalog contains

Search with `tools_search` and confirm with `tools_get` before every call. The committed catalog contains `integrations.credentials.custom_http_attach` and does not contain a capability to apply, list, update, or remove a custom HTTP manifest, and it does not contain `custom_http.<slug>.<operation>` tools. Do not invent those ids. If `integrations.providers_list` does not name a custom HTTP provider with a connect capability, say that registration is not available and stop. `integrations.integrations.upsert` writes a campaign provider row. It does not authenticate and it is not a manifest apply.

`integrations.health_get` is CRM sync health. It does not describe a custom HTTP API.

Absent controls, stated so they are not promised: manifest apply, manifest list, manifest remove, provider-side key revocation, and a catalog operation id generated from a slug. Detail on the shape you may still document for the user, without executing it: [manifest.md](manifest.md).

## Attach a credential

`integrations.credentials.custom_http_attach` replaces the encrypted credential for one integration. Input is `slug` (lowercase letter, then letters, digits, or underscores, at most 63 characters) and `secret_lease_id`. The secret arrives through the owner-only one-use lease. Raw credential fields never enter canonical params. The call is irreversible. It returns the slug and attachment state. Confirm the result does not echo the secret.

Never put a token, password, or header value in chat, a worksheet cell, a manifest draft, or ordinary run input. If the lease transport is unavailable, stop and say the integration was not connected. Do not ask the user to paste the secret into the conversation.

## If a worksheet HTTP column already exists

Authoring a new HTTP column is not this skill's attach call. `tables.columns.add` adds a manual column or an AI column with a prompt. It is not a manifest register. When a column binding is already on the worksheet, price it with `tables.table_runs.preview_cost` and read `estimated_credits`, `estimated_rows`, and `expires_at`. Run one row with `tables.columns.run` or `tables.table_runs.create` under the paid-run consent rule. Read `tables.table_runs.get` before any larger selection. Custom HTTP, if a binding exists, can still cost money at the upstream API even when workspace credits are zero. State that and get a ceiling before a live call.

A preview never approves a live write. Do not run a live request only to prove a credential attached.

## effect_unknown

`tables.table_runs.effect_unknown` lists provider receipts still calling or with an unknown outcome for one `table_id` and `run_id`. It is read-only. It does not retry, settle, release credits, or contact the provider. Default is unresolved receipts. `include_resolved` true adds receipts that already have reconciliation evidence.

Those calls are not safe to retry. Do not call `tables.table_runs.retry` on that run. Retry starts a fresh provider call and spends again. After the user verifies the provider-side result, settle with `tables.table_runs.reconcile_receipt` using the receipt id, the reconciliation outcome, and the preview the schema requires. A guessed success is not reconciliation.

Read `tables.table_runs.provider_summary` for held, captured, and released credits before you claim the run finished. Detail: [recovery.md](recovery.md).

## Safety ladder

1. `integrations.providers_list` and `tools_get` on `integrations.credentials.custom_http_attach`.
2. Lease plus attach. Read the attachment result.
3. If a column already runs that API: `tables.table_runs.preview_cost`, then one row, then readback.
4. Larger selection only after the one-row readback matches the expected output, under the paid-run consent rule.
5. On timeout or ambiguous status, `tables.table_runs.effect_unknown` before any other write.

## Recovery

| What happened | What to do |
| --- | --- |
| No manifest capability in `tools_search` | Report the limitation. Do not simulate apply with another integrations id. |
| Lease missing | Stop. Do not collect the secret in chat. |
| Attach result has no slug or still shows plaintext | Treat as failed. Do not retry with the secret in params. |
| `effect_unknown` returns receipts | Reconcile from provider evidence. Do not retry the run. |
| Preview quote and live refusal disagree | The refusal wins. A quote on a refused call is a price, not a confirmation. |
| Upstream error with a definite failure and no receipt in `effect_unknown` | Safe to consider a new one-row attempt after fixing inputs. Say which case you are in. |

## Worked example

Discover with `tools_get` on `integrations.credentials.custom_http_attach`. Attach:

```json
{"slug":"contact_api","secret_lease_id":"77000000-0000-4000-8000-000000000005"}
```

Use the lease id the secure transport actually returned. Read the returned slug and attachment state. If the body contains the secret, stop and report that plaintext came back.

When the user also has a worksheet column that calls the API, price with `tables.table_runs.preview_cost` for that column and a one-row selection. Read `preview_id`, `estimated_rows`, and `estimated_credits`. Run that one row under the paid-run consent rule. Read `tables.table_runs.get` for `status`, `completed_units`, and `credits`. If status is uncertain, call:

```json
{"table_id":"2b0f9c1e-7a3d-4c88-9f10-0b6a5d4c3e21","run_id":"4f1c2b64-7ad7-4b0a-9d0a-3d0f0d0d1a11","limit":100}
```

on `tables.table_runs.effect_unknown`. If `receipts` are present, stop. Show receipt ids. Do not retry.

## Decisions

- Discovery before promises. Absent ids stay absent.
- One-use lease only. Irreversible attach is not a dry run.
- One row before a batch, and only when a real column binding exists.
- Unknown provider outcome is not a failure you can safely repeat.
