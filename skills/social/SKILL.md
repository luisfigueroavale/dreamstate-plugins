---
name: social
description: "Own a short-form social post for its whole life across connected channels. Read the channel registry and learned brand voice, draft and revise, submit for review, approve, schedule or reschedule, publish now, unschedule, and archive."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. A larger paid run also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# Social

## Boundary

- Owns: short-form social post lifecycle, review, and publishing.
- LinkedIn post copy: [`linkedin-posts`].
- X post and thread copy: [`x-posts`].
- Reddit post copy: [`reddit-posts`].
- Articles and landing copy: [`writing`].
The `social` pack lists the tools and their exact inputs.

## Delete and archive

When user says delete a social post or draft, delete it permanently from Dreamstate. Show deletion impact first unless user already explicitly asked to delete. Delete removes local drafts, scheduled deliveries, and published-post records. Scheduled posts are unscheduled first. It never deletes provider posts from LinkedIn or X. When user says archive, archive only. Never say deletion is unavailable.

## Draft posts

One post is one draft that can go to several channels and several accounts at once. When the user asks for the same post on LinkedIn and X (or Reddit), create one draft whose channels list holds every channel, and put per-channel copy in that channel's payload: an X thread, a LinkedIn body, a Reddit title and subreddits. Never create one draft per channel for one request. A channel can carry several accounts (two LinkedIn profiles, or a company page and a personal profile); bind the accounts the user named, and when they said "all accounts" bind every connected account for that channel.

Before drafting or scheduling, inspect connected accounts for every requested platform and whether per-post auto-engagement is available. When more than one account is connected for a platform and the user did not name an account or say all accounts, ask which named account should post. When post request is vague, also ask what type to make, offering concrete choices such as story, how-to, opinion, announcement, or carousel. When auto-engagement is available and user has not decided it, ask whether to turn it on. Put every unresolved choice in one concise question with named account and post-type choices, never an open-ended prompt. A specific request with one connected account proceeds without questions, including no auto-engagement question.

A content plan is a set of drafts, each with its channels and an optional planned date. There is no weekly plan object and no workbook or other container for a plan.

Read the saved Company Context first, and the account's brand voice when one exists. When there is no account or no brand voice, write from the Tone of Voice page and say so. A missing account never blocks drafting.

Write with the channel skills: `linkedin-posts`, `x-posts` and `reddit-posts` hold the craft for each channel (hooks, structure, length, post types, closes). Read the one for every channel on the draft before writing its copy. The workspace can edit them, so follow the workspace copy.

Each request gets a new draft. Revise an existing draft only when the user names it or asks to change it.

Drafting needs no review, approval or human step. Match the stored voice. No em dashes in post copy.

Before scheduling, resolve every @mention for each channel. People and companies can be mentioned on LinkedIn and X. Never send unresolved mention text to a provider. It publishes broken public text and cannot be edited back on every channel.

Read the channel registry before writing. Per-channel max_content_length is registry data. Never assume a limit. X length is weighted (a link counts 23) and each thread post is checked on its own. Reddit is draft only: write the title, body and subreddits; the user posts it.

For LinkedIn, the last line must open with an ask verb or be a question addressed to the reader. Refuse the write with `linkedin_cta_required` when the close fails the rule.

## Automations

Per-post auto-engagement lets the workspace's connected accounts like, comment or repost after the post publishes. When the user asks for it ("add all our accounts as auto engagers"), do it: list the connected accounts, bind every account they named (or all of them), pick sensible defaults and save. Defaults: likes from every account; one comment per account, written by you in that account's voice, specific to the post and different for each account, never generic praise; stagger each account a few minutes apart instead of all at once. The post's own author does not like or comment on its own post. Report the accounts, actions, delays and exact comment copy you saved, so the user can change any of it.

Keyword auto-DM rules attach to a post. A comment containing the keyword gets a DM to the commenter and can get a public reply. Write the DM and reply copy yourself from the post and the offer; keep the DM short and deliver what the post promised.

Amplification is a standing workspace policy that engages from other people's real accounts and spends credits. Create it disabled, and arm it separately after scope, participant accounts, actions and credit ceiling are set.

These automations run only when the post publishes. Setting them up sends nothing.

## Schedule and publish

Scheduling is separate from drafting: schedule only when the user asks. Publishing is outward delivery. Decide by judgment whether to confirm first; when the user already asked to publish, publish. If the server refuses and names an approval it needs (a governed workspace's reviewer pipeline, or consent for a paid or recurring run), get that approval from the person; never invent one.

Resolve one explicit IANA timezone. Convert requested local time once into one timestamp for every channel. Never reinterpret “morning” per channel.

Read the saved draft, schedule or delivery result before reporting it done. When you change one channel of a post, send only that channel's copy, then read the post back and compare every other channel with what it was before your change; if any channel lost or changed copy you did not mean to touch, restore it before you report. Never say another channel is unchanged without reading it back. Never claim a result from the request alone.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
