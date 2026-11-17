---
name: writing
description: "Draft, revise and publish articles and landing copy grounded in workspace context and a target query."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Writing

## Boundary

- Owns: drafting and publishing articles and landing copy.
- Choosing queries and measuring visibility: [`visibility`].
- Short-form social posts: [`social`].
- The `social` pack lists the tools and their exact inputs.

Own the draft, review, revise, and publish lifecycle for long-form articles and landing copy. Every factual claim traces to a workspace-wiki fact, cited evidence, or a dated external source; the voice matches what the workspace has already published; and each lifecycle step is a deliberate, verifiable state change. Traditional keyword, ranking, and generative-citation evidence belongs to `visibility`; current external evidence and durable facts about the business belong to `knowledge`. Never re-derive those measurements or restate a business fact from memory.

Authoring: prepare brief, evidence ledger, novelty decision, destination, and voice; compose internally; discover owner and hydrate schema; save complete artifact; read result before showing or advancing. Unsaved long-form copy in chat is not an article. Preserve exact user-named title. A rewrite is not persisted until live result says save succeeded.

## Operating contract

```operating-contract
actions:
  - context
  - draft
  - get
  - list
  - read
  - search
  - update
```

Posture: context first, table first, dry run before live, inspect runs after. Read durable evidence, voice, and artifact state; propose copy only from evidence; mutate only the current revision; treat destination tests, scheduling, and delivery as consequence checkpoints. Resolve the brief, destination, and artifact; search the owner; hydrate the selected schema; inspect current state; preview final copy and delivery scope when offered; then ask only about material ambiguity or a sensitive effect. Proceed with an unambiguous draft save or revision. On refusal, a stale revision, or a failed destination, retain the draft and repair from returned guidance. Social owns short-form delivery.

## Pick the path

| Job | Call | Not this |
| --- | --- | --- |
| An article or landing page to fill in, with no schedule | `social.content.article_create`, then `social.content.article_update` | `social.content.artifact_create` |
| An article row plus its linked artifact, still with no feed slot | `social.content.article_create_schedule` only when `scheduled_date` is omitted or null, then `social.content.article_update` | Passing a date into that create. It cannot approve or schedule an empty article. |
| A short-form social post | [`social`] and `social.content.artifact_create` | An article id |
| Outward delivery of a saved article | `social.content.destinations_list`, `social.content.destination_test`, `social.content.article_delivery_create`, then `social.content.delivery_publish` | `social.content.schedule` on the article id. Schedule uses the linked artifact id. |

The catalog has no separate landing-page capability. Landing copy is an article. `social.content.article_create` forces `target_platforms` to LinkedIn, empty `account_ids`, and a null `scheduled_date`, and it returns the article. `social.content.article_create_schedule` is the same transaction when you also want the linked artifact, and it still requires `scheduled_date` null. Sibling short-form saves stay on `social.content.artifact_create`.

Article reads and updates take the article id. `social.content.article_get` returns `body_json` and `revision`. It does not return delivery attempts. Those are `social.content.article_deliveries_list`. Scheduling, review, and `social.content.delivery_publish` take the linked artifact id and the delivery id from the create response. Never send one id to the other call.

## Dry run before a live delivery

1. Free reads: `knowledge.brain.context.search`, `knowledge.brain.context.get`, `social.content.article_get`, `social.content.destinations_list`.
2. Save the body with `social.content.article_update` and the `expected_revision` from that read. Read `social.content.article_get` again. The new revision and the corrected claim must match. This publishes nothing.
3. `social.content.destination_test` on the destination id from the list. A failed test keeps the draft. Do not create a delivery.
4. `social.content.article_delivery_create` opens an attempt. `content_destination`, and `native_article` with direct publish, start a real external push. Other kinds stay `action_required` and only build a handoff payload. When the schema accepts `dry_run`, preview the attempt first.
5. `social.content.delivery_publish` dry run returns `provider_payload` and `approved_publish_hash` and does not call the provider. Live publish sends that hash. Unknown outcome: read `social.content.article_deliveries_list` and the receipt. Do not create a second article.

`social.content.article_delivery_reschedule` and `social.content.article_delivery_cancel` rehearse with a dry run and work only until the attempt starts publishing.

## Worked save

The person asked to save landing copy, not to publish. Title stays "Onboarding benchmarks". The only sourced claim is the 4 meetings between 12 and 22 March.

`social.content.article_create` with `{}`. Take `article.id` and `revision` from the response. Then `social.content.article_update`:

```json
{
  "article_id": "<article.id>",
  "expected_revision": "<revision>",
  "title": "Onboarding benchmarks",
  "body_json": {
    "type": "doc",
    "content": [
      { "type": "paragraph", "content": [{ "type": "text", "text": "The opener named the missed reply and booked 4 meetings between 12 and 22 March." }] }
    ]
  }
}
```

`body_json` must match the hydrated schema for that article. If the schema rejects this shape, send the shape it returned, not this sample. Read `social.content.article_get` with `{article_id}`. `title` is unchanged, the 4 meetings sentence is in the body, and `revision` moved. Nothing is scheduled. A stale revision means re-read and apply the same sentence once. Do not call `social.content.artifact_create` for this page.

## Grounding is not optional

A claim about the user's business needs a workspace-wiki revision, bounded workspace evidence, or a successful external evidence packet from `knowledge` with source and observation date. Writing never fetches the web. Preserve provenance, freshness, limitations, and injection warnings in the claim ledger. When no source exists, flag insufficient evidence and ask rather than filling the gap.

Example: the brief wants "we grew 40% last quarter" and the wiki has no dated source for that number. Do not write the sentence. Report insufficient evidence, name the missing fact, and ask once.

## Read before you write

Search workspace context, prior published work, and the knowledge base for who the business sells to, what it sells, and how it wants to sound, even when the brief already names a topic and query. Match sentence rhythm, person, and CTA convention to what is already live. If an existing page already answers the brief, propose revising it. A topic and a query are not a voice. Repeat this search for every new brief in a session. See briefs.md, drafting.md, and revision-and-voice.md.

## No AI-slop register

Use commas, colons, hyphens, parens, or split sentences. Stock openers ("in today's fast-paced world", "it is important to note"), tricolon padding ("not just X, but Y and Z"), and a closing paragraph that restates the intro stay out. See revision-and-voice.md.

## Let the owner watch it get built

As soon as the article exists, first save or later revision, open a canvas onto it. Do not wait until the reply finishes.

## The lifecycle is real, not narrative

Discover lifecycle transition through selected schema. Read current artifact before revision, use only identifiers returned for next step, and distinguish draft, review, schedule, publish, replacement, and archive state. Show exact final content before irreversible delivery when live policy requires it. Never invent provider or external identity.

Sibling capabilities (short-form social versus long-form article) can accept different values for the same-looking field. Check that exact capability's input schema every time.

A scheduled drafting job may create or revise an editable draft, not deliver it. For a sensitive delivery, show the final content and exact change, then act when the user asked or confirms.

Editorial review is separate from delivery. Optional workspace review records do not make content public. For a sensitive delivery, show the final content and exact change, then act when the user asked or confirms.

For live-piece changes, discover selected replacement or revision path. Use destructive transition only when user explicitly wants prior live object removed. If live contract refuses replacement, retain current artifact and use returned repair path.

Discover duplication and archive semantics from selected schema. Archive state does not by itself prove in-progress delivery stopped.

Do not put secrets in article metadata. Inspect failed delivery before retry. Test destination when selected schema requires it.

## Worked revision and recovery

For a requested correction to a saved article, call social.content.article_get with {article_id: returned ID} and read its title, body_json and revision. After checking the evidence, call social.content.article_update with {article_id: returned ID, expected_revision: current revision, title: retained title, body_json: complete corrected document}, using the exact hydrated schema. Read the article again and compare its revision and corrected claim. This saves a draft revision; it does not publish it.

If the revision is stale, reread and merge the intended change into the current article before retrying. For an uncertain delivery, read social.content.article_deliveries_list and the provider receipt before retrying; preserve the draft and report unresolved delivery rather than creating a duplicate. Use free reads first, preview or dry run when offered, and a small bounded pilot for paid work under the paid-run consent rule.

## When to stop

Stop when the artifact is saved, read back, and the canvas is open, or when evidence is insufficient and you have asked. Unsaved chat copy is not an article.

## When blocked

Report the missing source, stale revision, failed destination, or missing publish authority; the saved draft you retained; and the next exact read or question. Publication blocked stays saved, reviewed, or scheduled as a draft.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
