# Publishing lifecycle

## Two id spaces: article vs artifact

An article has an article id and a linked content-artifact id. Article reads, updates, delivery creation, duplication, and archive use the article id. Scheduling, unscheduling, editorial review, optional workspace review records, replacement, and delete-and-republish proposals use the linked artifact id. Read the article first and take both identities from the response. Never infer that two operations accept the same id because they act on the same piece of content.

For every call, search the content catalogue for the intended outcome and read its exact input schema. The schema, not this prose, decides which id and fields the operation accepts.

## Platform values differ by family

Article creation, article updates, and native-article deliveries accept linkedin or twitter. Short-form social operations use linkedin or x. These are separate enums. Match the literal to the discovered contract and never assume one universal platform name.

## The state machine

1. Search the article surface for the operation that creates an article with its initial schedule. It creates the article, linked artifact, first version, and delivery scaffolding in one transaction. There is no separate bare-draft article creation path.
2. A scheduled drafting job may create or revise reviewable drafts, but it never delivers content. For outward delivery, show the exact final revision and destination, then act when the user asked or confirms.
3. Revise through the article update operation using the current expected revision from an exact article read. On revision conflict, re-read and deliberately reapply. An in-flight delivery or slug collision is a state conflict to inspect, not a reason for blind retry. Treat prior review records as historical after any edit.
4. Editorial submission moves the linked artifact into review. It does not schedule or deliver.
5. Scheduling and unscheduling act on the linked artifact id. Scheduling is reversible until delivery fires.
6. Before creating a delivery, list configured destinations and test the chosen destination's current readiness. Never create a delivery against an unchecked destination.
7. Delivery creation creates an attempt, not a publish. Native articles may be direct only when destination scopes allow it; content destinations are asynchronous; external copy is handoff-only. Reuse an idempotency key only for the exact same retry payload.
8. Outward publishing uses the delivery id returned by delivery creation. It is irreversible. An explicit trigger may bypass scheduled time and publish immediately, so use it only for an immediate-publish request. Show the exact final content before execution and report the returned provider receipt without constructing any identifiers.
9. After publish, do not assume the article remains editable. If an update is refused or editability is uncertain, use the replacement path described below.

## Optional workspace review records

Two distinct review-record models may be available:

- A simple artifact lifecycle review marks editorial state while leaving the artifact editable. It does not make content public.
- A named-reviewer workflow records submission and individual decisions. It is an audit trail for workspaces that choose formal sign-off.

Discover the chosen model's operations from the catalogue. Neither model performs outward delivery, and outward delivery does not imply that either model ran. If the user or workspace chooses review records, record them accurately. Never treat them as a delivery gate. If no review policy applies, do not invent one.

## Replacing already-published content

For a live piece, search the content surface for replacement creation. The normal replacement mode creates a separate proposed artifact linked to the original and leaves the original provider object intact. Delete-and-republish mode also removes the old provider object where supported and loses its engagement continuity, so use it only when the user explicitly wants the old version removed.

A replacement never publishes itself. It requires a fresh delivery attempt and outward publish. Before the original has a published delivery, replacement creation must refuse. For an unpublished draft, retrieve the desired historical version and diff, then deliberately apply that content through the article update operation. Do not assume a one-call rollback exists.

## Reading state without changing it

Search the content surface for exact article state, article lists, enabled state, delivery history, delivery payload, and distribution reads. Use these before answering status questions and before choosing update, replacement, or new delivery. A search result or requested mutation is not proof of persisted state.

## Versioning is explicit

Article updates do not automatically create snapshots. Create a version boundary deliberately before a substantial rewrite, before publish, or at the user's request. Use version listing, exact-version reads, and explicit diffs to inspect history. Never claim rollback from a diff alone; apply the chosen historical content through a current-revision update and read it back.

## Duplicate, archive, and unschedule

Duplication clones the article and its assets and is protected by an idempotency key. Archive removes an article from active circulation. Unscheduling withdraws a pending delivery. They are different decisions: archiving does not prove an in-flight or scheduled delivery stopped.

## Metadata safety

Publishing metadata and schema fields reject secret-shaped keys and prototype-pollution keys. Do not rename a credential-like value to evade validation. Secrets do not belong in publishing metadata.

## What makes the work complete

- The article readback proves the exact article and linked artifact identities.
- The final revision and body match what the user reviewed.
- Destination readiness was checked before delivery creation.
- Scheduling reports the intended instant, or outward publishing returns a terminal provider receipt.
- Review state is described as policy state, never as proof of delivery.
- Replacement state clearly distinguishes the proposed replacement from the still-live original.
- Unknown delivery outcomes are resolved from durable delivery state before any retry.

## Traps with real consequences

- Using an explicit delivery trigger for a schedule-only request, which publishes immediately.
- Treating review as scheduling or publishing.
- Passing the article id to an artifact-scoped operation.
- Retrying delivery creation with a new idempotency key before investigating the original attempt.
- Skipping destination readiness checks.
- Attempting replacement before a published delivery exists.
- Reporting a requested mutation as complete without an exact article or delivery readback.
