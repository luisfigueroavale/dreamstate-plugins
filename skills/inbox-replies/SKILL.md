---
name: inbox-replies
description: "Triage and answer existing LinkedIn and email conversations, classify the latest inbound message, confirm the sequence stopped, and draft a reply grounded in the real thread and Company Context. Use it when someone replied, when the user wants to work through unanswered threads, or when a positive reply should move toward a meeting. It does not start new conversations with people who have not been contacted (the outreach skill does) or build automated reply handling (the workflows skill does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# Inbox Replies

## Boundary

- Owns: replies in existing email and LinkedIn conversations.
- New conversation or cadence: [`outreach`].
- Records and pipeline state: [`crm`].
- The `unibox` pack lists the tools and their exact inputs.

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

Search the inbox tool for the task. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run.

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
