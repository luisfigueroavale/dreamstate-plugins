---
name: writing
description: "Draft, revise and publish articles and landing copy grounded in workspace context and a target query."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run` (`dry_run` first for anything that writes, spends credits or reaches a person). If the server asks you to sign in, approve it in the browser.

# Writing

## Boundary

- Owns: drafting and publishing articles and landing copy.
- Choosing queries and measuring visibility: [`visibility`].
- Short-form social posts: [`social`].
- The `social` pack lists the tools and their exact inputs.

<!-- architect-operation-contract
{"required_capability_ids":[]}
-->

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

## When to stop

Stop when the artifact is saved, read back, and the canvas is open, or when evidence is insufficient and you have asked. Unsaved chat copy is not an article.

## When blocked

Report the missing source, stale revision, failed destination, or missing publish authority; the saved draft you retained; and the next exact read or question. Publication blocked stays saved, reviewed, or scheduled as a draft.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
