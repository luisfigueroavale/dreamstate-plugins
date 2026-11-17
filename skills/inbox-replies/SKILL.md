---
name: inbox-replies
description: "Triage and answer existing LinkedIn and email conversations, classify the latest inbound message, confirm the sequence stopped, and draft a reply grounded in the real thread and Company Context. Use it when someone replied, when the user wants to work through unanswered threads, or when a positive reply should move toward a meeting. It does not start new conversations with people who have not been contacted (the outreach skill does) or build automated reply handling (the workflows skill does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Inbox Replies

## Boundary

- Owns: replies in existing email and LinkedIn conversations.
- New conversation or cadence: [`outreach`].
- Records and pipeline state: [`crm`].
- The `unibox` pack lists the tools and their exact inputs.

A single direct email outside a thread uses `sequence.outreach.email_send_one_off`. A new LinkedIn conversation belongs to [`outreach`].

A reply is the whole point of outreach. Answer what the person actually said, in the conversation they said it in, and never restart the pitch.

## Backing packs

The unibox pack owns mailboxes, conversations, replies and conversation state, and lists the tools for this skill. Sequence run state (whether the cadence stopped) lives in the sequence pack. Business facts come from the knowledge pack, read the way the knowledge skill describes.

## The invariant

An existing conversation is answered in that conversation. Before any reply goes out, the user sees the exact channel, account, recipient and final copy. A new conversation with someone never contacted is a sequence, even for one person and one message.

## Practice

1. **Take a bounded human-reply slice.** Pull only `needs_human_reply: true` for one channel and a time window. Do not load structural `unanswered` work: vendor pitches, thank-yous, reactions, declines, out-of-office messages, and closed threads do not need a human reply.
2. **Read the full thread.** Every message in both directions, the person's worksheet row, then the Company Context pages the answer depends on: Product Information for facts, Tone of Voice for register.
3. **Classify the latest inbound.** Interested, question, objection, not now, wrong person, referral, unsubscribe or out of office. Decide from the words in context, not a keyword match. When it is ambiguous, say so.
4. **Check the stop from the conversation read.** The conversation read returns `sequence_stop_state` with `state`, `evidence`, and `observed_at`. Confirm `state: stopped` before drafting. `active` or `unknown` is not confirmation and must be resolved before any send.
5. **Draft the answer.**
   - Interested: one short paragraph, a concrete next step, a specific way to book.
   - Question: answer from Company Context, cite the page, then one ask.
   - Objection: acknowledge it in their words, one proof point with a source, one low-effort ask.
   - Not now: agree, ask permission to follow up at a stated time.
   - Wrong person or referral: thank them, ask for or confirm the right name.
   - Unsubscribe or decline: no pitch. Confirm suppression.
6. **Show, then send.** Present channel, account, thread, recipient and final copy. Send when the user confirms or has already asked you to send this reply.
7. **Read back.** Confirm the message appears in the conversation before reporting it sent.

## Routing and evidence

Use `unibox.inbox.unified_list` for a bounded conversation slice, `unibox.inbox.messages_query` for stored message search, and `unibox.inbox.conversation_get` for the exact account-scoped thread. Page older messages. Counts describe the authorized filters, not necessarily the entire mailbox. Triage status expresses sales intent: interested can include a thank-you, and a new reply reads neutral until analyzed. `unibox.inbox.analyze` can cost credits; preview its quote and choose the mailbox scope before running it. Do not mistake neutral for lack of interest. A reply raises the record's CRM signal level; `crm.leads.today_list` helps find today's leads.

| Latest message | Decision |
|---|---|
| A real question, objection, referral or interest | Answer the specific point and offer one appropriate next step. |
| A request to revisit later | Agree on a follow-up date; send a booking link only when asked. |
| A vendor pitching you, a thank-you, a reaction or "will check it out" | Usually no reply. Review the context before closing. |
| A decline, unsubscribe or out-of-office | Do not pitch; confirm suppression when applicable and defer an out-of-office. |
| A partner or agency pitch that merits closure | Give a brief, courteous closing line rather than a demo pitch. |

For a thread delayed three or more days, acknowledge the delay. Keep short direct-message drafts under 240 characters where practical, with one next step and no sign-off. Avoid the default demo closer, stock closing questions, feature dumps and references to private profile views. Never invent a fact.

Use `unibox.inbox.reply_draft_edit` to save a draft and `unibox.context.thread_reply_send` to reply in the existing thread. Read the exact schemas with `tools_get`. Preview the send, show the exact content and destination, and follow the paid-run consent rule, including any applicable approval and ceilings. Start with one reviewed reply before any broader batch. Re-read the thread immediately before sending; a new inbound invalidates the draft.

Triage, tags, and archive are separate from the reply itself. State the count before a bulk change.

- Triage a reviewed set with `unibox.conversations_bulk_triage`. Preview in dry run. Pass `conversation_ids` (at most 100 exact account-scoped ids) and one action. `reply_label_set` plus `label` (`interested`, `question`, `objection`, `not_interested`, `ooo`, `vendor_pitch`, `closed`, or `unclear`) sets the label on each conversation's latest inbound reply. `unibox.inbox.analyze` can persist a classify-only triage and may cost credits, so preview its quote first. Classification in this skill is still a reading of the words, not a keyword match.
- Tags on one conversation use `workspace.tags.list` with `object_type` `conversation`, then `workspace.tags.attach_delta` with `object_type` `conversation`, `object_id`, and `add` or `remove`. `workspace.tags.attach` replaces the whole set. Inbox labels on a set use the same bulk triage call with `labels_add` or `labels_remove` and `labels`. Those labels are not workspace tags.
- Archive a conversation with `unibox.conversations_bulk_triage` action `archive_set` and `archived` true. That is not `unibox.inbox.status_label_archive`, which retires one custom status label and refuses `system_label` and `label_in_use`. Set one conversation's status with `unibox.inbox.conversation_status_apply` (`conversation_id`, `label_key`) after `unibox.inbox.status_labels_list`.

## Recovery

Walk this ladder in order. Never mint a new request id while an earlier send might have landed.

1. Refused send. The provider was not contacted. Read the refusal. Fix the missing approval, the stale `expected_revision`, or the ceiling, then send again with the same `client_request_id` only when the refusal says nothing was delivered.
2. Unknown outcome. Do not send again. Read `unibox.inbox.conversation_get`. If the outbound message is already in the thread, stop.
3. Persisted draft. `unibox.inbox.reply_draft_edit` saves a draft and does not send. Read it back with `unibox.inbox.reply_drafts_list` filtered to that `conversation_id`. A send receipt with `persisted` true has a run. `persisted` false returns `not_found` from `runs.get`.
4. `runs.get`. When the receipt has `run_id` and `persisted` true, read `runs.get` and reconcile that run. `not_found` is not proof the message is absent if the conversation already shows it.

## Worked example

An email says, "Interested, but next month is better." Read `unibox.inbox.conversation_get` with `{"conversation_id":"email:11111111-1111-4111-8111-111111111111:thread-1"}` and confirm the latest inbound and stopped sequence. Draft: "Happy to reconnect next month. Would December 10 work for a follow-up?" Preview `unibox.context.thread_reply_send` in `dry_run` mode with `params` containing `conversation_id`, `content: {"body":"Happy to reconnect next month. Would December 10 work for a follow-up?","attachments":[],"voice":null}`, and a stable UUID `client_request_id`. Show the destination and copy, then send only when authorized, using the same operation identity. Read the conversation again and report the stored message or the unresolved outcome. Do not add a booking link unless requested.

## Guardrails

- Never turn a reply into a new sequence enrollment, and never reach the same person through another sender or channel after a stop.
- A conversation status is not proof of delivery. After an unclear send result, read the conversation before sending again. Never send twice.
- Do not promise what Company Context does not support (discounts, features, dates). Ask the user.
- Bulk changes (tags, archive) state the count first.
- A lasting fact the thread reveals (a new objection pattern, a competitor named) is offered for filing to the right Company Context page.

## What to return

- Per thread: person, channel, classification with the quoted line that decided it, sequence stop state.
- The draft reply and the pages it cites.
- After sending: confirmed in the conversation, with a link. Otherwise the state proven and what is left.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
