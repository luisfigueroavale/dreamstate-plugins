---
name: engage
description: "Reply to comments and mentions on your own LinkedIn, Instagram and X posts, watch X lists and LinkedIn profile lists to comment on their posts, find X posts by keyword, and track your comments and the responses. It does not write or schedule your own posts (social does) or answer DMs (inbox-replies does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Engage

## Boundary

- Owns: the Engage queue on the user's own LinkedIn, Instagram, and X accounts. The tabs are Mentions, Lists, and Discover. Comments on the user's posts, mentions, and the user's own comments (item type `replies_to_my_comments`) all live in Mentions. There is no Comments tab.
- Writing or scheduling the user's own posts: [`social`]. Engage does not draft or schedule that post.
- DMs and inbox threads: [`inbox-replies`]. An item's `actions` can show a keyword DM or a DM state. Sending that DM still belongs to inbox replies.
- X reply copy craft: [`x-posts`].
- Turning engagers into leads: [`linkedin-warm-signals`].
The `engage` pack lists the tools and their exact inputs.

## How to work a queue

1. Call `engage.accounts.list` first. Each account says which tabs it supports, how much of today's action budget is left, `reply_as_options`, and `can_upload_media`.
2. List open items for the tab the user means. Mentions is the default. Lists and Discover are the other tabs.
3. Pick what deserves a reply (below), draft, preview, approve. Dismiss spam. Leave the rest open.
4. Finish with what was replied to, what was dismissed, and what still waits.

## What each network supports

- Mentions: comments on the user's own LinkedIn and Instagram posts, X mentions plus replies to the user's X posts, and the user's own comments (`replies_to_my_comments`). LinkedIn and X come from the network. For the user's own Instagram comments, Dreamstate shows only replies it sent.
- Each item carries `actions` (keyword DM, Engage reply, DM, and reaction states), `permalink` (open the post on the network), `post_author`, and `my_comment`.
- Lists: an X list the account owns, or one public X list by id or URL. Followed X lists are not on our plan. LinkedIn lists of up to 50 profile or company URLs. Instagram has no lists: say so and do not create one.
- Discover: X keyword search only.
- Reply as (`reply_as_options` on `engage.accounts.list`): LinkedIn, any active LinkedIn member or page. Instagram, any active Instagram account. X, only that item's own account.

## When to use which capability

- `engage.accounts.list` before listing items or previewing. It returns `reply_as_options` and `can_upload_media`.
- `engage.lists.x_available` is the read for owned X lists. Input is `{ account_id }`. Followed lists are unavailable on our plan: X returns 403, title "Client Forbidden", reason `client-not-enrolled`. The read reports that. It does not fail the owned lists.
- `engage.lists.import` is the write that imports one list by id or URL. It needs an idempotency key. An X list can be imported from a list URL or from a list the account owns. Do not call this write to list available lists: that call is refused `400` `idempotency_key_required` before X is called.
- `engage.replies.preview` drafts the public reply. Pass `reply_as_account_id` for LinkedIn (any active member or page) or Instagram (any active Instagram account). X refuses with `reply_as_not_allowed`.
- Media on that preview follows `can_upload_media`: LinkedIn, one image. X, up to 4 images when the token can upload. Instagram, none. The X compose fallback carries text only, so it gets no media.
- `engage.reactions.add` is LinkedIn only, and the call must set `approved` true.

## Freshness

There is no live push. Comment and mention checks run at most once an hour, only inside the account's working hours, and a post stops being checked 14 days after it was published. LinkedIn reads count against the shared daily action budget. Replies are re-read only when reply counters move. LinkedIn lists refresh once a day at 12:00 workspace time. Say this plainly when the user expects real time.

"Paused, daily limit" means the account's daily action budget is spent. It resets at midnight in the workspace timezone. Do not sync or reply on that account until then.

## Choosing what to reply to

Reply first to questions, people who fit the ideal customer, authors with a large following, and comments nobody has answered yet. Never reply to every comment: a feed of identical thank-yous reads as a bot. Dismiss spam without drafting.

## Writing the reply

Read the comment and the post it sits under before drafting. Use the user's voice from Company Context. One to three sentences, specific to what the person said: answer the question, add one concrete detail, or ask one real follow-up. No generic praise ("Great point!"), no hashtags, no links unless the user asked.

## Sending

Always preview, then approve with the content hash the preview returned. Any change to the text needs a new preview, so an old hash is never reused. Never retry a reply whose outcome is unknown: tell the user to check the thread first. An `effect_unknown` reply is never retried, and a second approve is not a recovery.

Public writes wait for approval. A reply uses the preview hash. A reaction sets `approved` true on `engage.reactions.add`. A list import sends an idempotency key. Do not send any of them on your own.

An X list post cannot be replied to through the API when the author did not mention the account. X allows an API reply only when the original post's author mentioned or quoted the replying account (self-serve rule since 2026-02-23). Otherwise X rejects it. Imported list posts and Discover posts are for reading and for an open-in-X draft, not for an API reply unless the author mentioned the account (or quoted the replying account). Hand the user the open-in-X link (the x.com compose intent) together with the draft, then mark the item done. That intent carries text only (no media). No API reply is sent.

## What not to do

- Do not look for a Comments tab. The user's own comments are Mentions items of type `replies_to_my_comments`.
- Do not list X lists with `engage.lists.import`, and do not promise followed lists.
- Do not reply as another X account. Do not attach media on Instagram, on the X compose fallback, or on X when `can_upload_media` is false.
- Do not add a reaction except on LinkedIn, and never without `approved` true.

## Budgets

Every comment check and reply spends the account's daily actions. LinkedIn shares one daily cap with outreach. Instagram counts total actions per day and per hour, DMs included. A manual sync spends the same budget. When a reservation is refused, stop and say which account is paused and until when.

## Check it works

Read the reply result before saying it was sent. An unknown outcome is neither sent nor failed. Report each reply in plain words: who it answered, on which post, and whether it went out or was handed over as an open-in-X link.

Paid and outward runs follow the paid-run consent rule.
