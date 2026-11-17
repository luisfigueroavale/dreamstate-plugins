---
name: social
description: "Own a short-form social post for its whole life across connected channels. Read the channel registry and learned brand voice, draft and revise, submit for review, approve, schedule or reschedule, publish now, unschedule, and archive."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Social

## Boundary

- Owns: short-form social post lifecycle, review, and publishing.
- LinkedIn post copy: [`linkedin-posts`].
- X post and thread copy: [`x-posts`].
- Reddit post copy: [`reddit-posts`]. Articles and landing copy: [`writing`].
The `social` pack lists the tools and their exact inputs.

## Published-post scoring

For published-post review, find the published-post score read for one post or the published-post score list for a date range. Scores compare each author account with its own same-platform recent baseline at 24h and 7d. A `not_enough_history` label means fewer than three prior posts, not poor performance. These reads use stored snapshots and never refresh a provider.

## Delete and archive

When user says delete a social post or draft, delete it permanently from Dreamstate. Show deletion impact first unless user already explicitly asked to delete. Delete removes local drafts, scheduled deliveries, and published-post records. Scheduled posts are unscheduled first. It never deletes provider posts from LinkedIn or X. When user says archive, archive only. Never say deletion is unavailable.

## Draft posts

One post is one draft that can go to several channels and several accounts at once. When the user asks for the same post on LinkedIn and X (or Reddit), create one draft whose channels list holds every channel, and put per-channel copy in that channel's payload: an X thread, a LinkedIn body, a Reddit title and subreddits. Never create one draft per channel for one request. A channel can carry several accounts (two LinkedIn profiles, or a company page and a personal profile); bind the accounts the user named, and when they said "all accounts" bind every connected account for that channel.

Before drafting, read any evidence on which audience and channel responded to past outreach, following the performance skill. Answer questions about how posts performed with `social.content.performance_query`, where engagement rate is likes, comments, reposts and saves over impressions for the same posts at the same age. Before drafting or scheduling, inspect connected accounts for every requested platform and whether per-post auto-engagement is available. When more than one account is connected for a platform and the user did not name an account or say all accounts, ask which named account should post. When post request is vague, also ask what type to make, offering concrete choices such as story, how-to, opinion, announcement, or carousel. When auto-engagement is available and user has not decided it, ask whether to turn it on. Put every unresolved choice in one concise question with named account and post-type choices, never an open-ended prompt. A specific request with one connected account proceeds without questions, including no auto-engagement question.

Before any batched create, resolve each target channel account once: use the only connected account, the account the user named, or ask. Never fan out creates with a missing required input.

A content plan is a set of drafts, each with its channels and an optional planned date. There is no weekly plan object and no workbook or other container for a plan. Never put weekday or date words in titles of unscheduled drafts.

Read the saved Company Context first, and the account's brand voice when one exists. When there is no account or no brand voice, write from the Tone of Voice page and say so. When an account is required to create a draft, resolve it before creating.

Write with the channel skills: `linkedin-posts`, `x-posts` and `reddit-posts` hold the craft for each channel (hooks, structure, length, post types, closes). Read the one for every channel on the draft before writing its copy. The workspace can edit them, so follow the workspace copy.

Each request gets a new draft. Revise an existing draft only when the user names it or asks to change it.

When a post you changed comes back different, read its version history before explaining it. Each version names its writer: `agent_edit` with the surface (MCP, CLI, Architect) or `manual_edit` from the dashboard. Report what the history shows. Never tell the user to refresh, close or reload tabs as the fix: an open editor reports a conflict instead of overwriting newer copy.

Drafting needs no review, approval or human step. Match the stored voice. No em dashes in post copy.

Before scheduling, resolve every @mention for each channel. People and companies can be mentioned on LinkedIn and X. Never send unresolved mention text to a provider. It publishes broken public text and cannot be edited back on every channel.

Read the channel registry before writing. Per-channel max_content_length is registry data. Never assume a limit. X length is weighted (a link counts 23) and each thread post is checked on its own. Reddit is a write-enabled channel on `social.content.artifact_create`. Put `reddit_title` and `subreddits` on the draft. A connected Reddit account publishes on the queue below. `social.sender_profile_reddit_upsert` stores a label only. It does not connect an account and it does not grant a post.

For LinkedIn, the last line must open with an ask verb or be a question addressed to the reader. Refuse the write with `linkedin_cta_required` when the close fails the rule.

## Media

To attach a file, request the post's `social.content.media_upload_url`, PUT the raw bytes to the returned `upload_url` with the returned headers, then call `social.content.media_uploaded`. If that returns a queued run, read the result with `runs_get` before telling the user anything. A file already at a public https URL in workspace storage can be attached by putting `{type, url, name, alt_text}` in the post's media array.

A failed upload states its reason. Fix the cause and request a new upload URL. Never tell the user a format is unsupported unless the limits below say so, and never swap one media type for another (for example a video in place of a GIF) without asking. If you cannot PUT the bytes, `social.content.media_import` downloads a public https file URL, up to 50 MB, and attaches it after the same validation. A page URL is not a file URL.

Media is replace-the-set on update: send the complete array each time. Array order is display order.

Per channel:
- LinkedIn: up to 9 images or GIFs, or 1 video (mp4 or mov), or 1 PDF document shown as a swipeable carousel, never mixed. An animated GIF posts as an animated GIF.
- X: up to 4 images, or 1 GIF, or 1 video. No documents.
- Instagram: up to 10 images as a carousel, or 1 video. A GIF is converted to video. A Story is exactly 1 item. No documents.
- TikTok: images or 1 video.
- YouTube: exactly 1 video.
- Facebook: images or video, up to 10.
- Threads: images or video, up to 20.
- Reddit: 1 image or 1 video. Title and subreddits still live on `platform_payloads.reddit`. Publishing needs a connected Reddit account, not a sender-profile label.

Read `social.channels_list` for current limits.

## Automations

Per-post auto-engagement lets the workspace's connected accounts like, comment or repost after the post publishes. When the user asks for it ("add all our accounts as auto engagers"), do it: list the connected accounts, bind every account they named (or all of them), pick sensible defaults and save. Defaults: likes from every account; one comment per account, written by you in that account's voice, specific to the post and different for each account, never generic praise; stagger each account a few minutes apart instead of all at once. The post's own author does not like or comment on its own post. Report the accounts, actions, delays and exact comment copy you saved, so the user can change any of it.

On X, auto-engagement runs only as the account that publishes the post, and you save the complete list each time (at most one of each action). Auto comment replies after a delay. Auto retweet retweets after a delay, and remove retweet undoes that retweet a set time after it happened (it needs the retweet in the same list). Auto repost can instead repost with a comment: it posts your comment with a link to the post, which X shows as an embedded post (text only, up to 256 characters). Auto comment and auto plug replies can carry up to 4 images, or 1 GIF, or 1 video from workspace uploads. Auto plug replies with a call to action once the post reaches a likes threshold, rechecked every 30 minutes until its watch window ends. Auto delete removes the whole thread when likes are still below a threshold after the delay; it cannot be undone, so turn it on only when the user asked for it and say so in your report. There is no auto-like on X. Each X account can store default automations that only pre-fill the editor. Super followers only is a per-post X option and needs Super Follows on the account. To let someone else connect their X account, email them an X account invite only when the user gave the address; the link works once and expires after 7 days.

Keyword auto-DM (comment-to-DM) rules attach to a post. Discover the rule read and replacement capabilities through the social pack, then read the current rules and replace them directly; there is no separate follow-up tool. The editor's "private follow-up" switch is `dm_enabled` with `dm_message`. On LinkedIn the DM reaches only 1st-degree connections: with the DM on, a connection gets the DM (and `comment_reply_sent` after it, when enabled) and anyone else gets `comment_reply_not_connected`, when enabled. On X the DM goes right away to commenters who follow the account and accept DMs, and everyone else gets `comment_reply_not_connected`, when enabled. New rules start with the DM off, as a plain comment reply. With the DM off, every matching commenter gets `comment_reply_sent` when it is enabled, and the not-connected reply is not used. A rule that reaches nobody is refused. Make the post's CTA promise what the rule delivers: "I'll DM you" only with the DM on. Write the DM and reply copy yourself from the post and the offer; keep the DM short and deliver what the post promised. Report the keywords and what a connected and a not-connected commenter will each receive.

Amplification is a standing workspace policy that engages from other people's real accounts and spends credits. Create it disabled, and arm it separately after scope, participant accounts, actions and credit ceiling are set.

These automations run only when the post publishes. Setting them up sends nothing.

## Comments and generation

Use social.comments.queue_list and social.comments.get to read owned-post comments and freshness. Use social.comments.update for local drafts or notes and social.comments.resolve for local triage. For a public reply, social.comments.reply freezes a preview; review its text and use social.comments.approve with its returned action_id and content_hash and approved true. An edit requires a new preview. If the outcome is effect_unknown, inspect the live thread and receipt before further action; never resend blindly. social.comments.x_refresh fetches X replies on demand and consumes provider reads.

For optional copy assistance, social.tools.linkedin_post_generate, social.tools.linkedin_hook_generate and social.tools.linkedin_headline_generate serve LinkedIn; social.tools.x_post_generate, social.tools.x_hook_generate and social.tools.x_bio_generate serve X. Inspect each schema and cost, ground and review its output, then save the chosen copy as an artifact. Generation alone does not prove publication.

## Pick the path

| Ask | Call | Not this |
| --- | --- | --- |
| A time, media, a thread, a poll, Reddit, or more than one channel | `social.content.artifact_create`, then `social.content.schedule`, `social.content.approve`, then `social.content.delivery_publish` | `social.posts.create` |
| LinkedIn text now, no media, no time, one account | `social.posts.create` preview, then live with `mode` `live` and `approved` true | A second artifact for the same text |
| Reply on a comment already in the owned queue | `social.comments.reply`, then `social.comments.approve` | A second approve after `effect_unknown` |
| A new public comment on a live post | `social.posts.comment_publish` under the paid-run consent rule | The owned-queue preview |

Worked params and readbacks are in [lifecycle.md](lifecycle.md).

## Confirm first

1. Free reads: `social.channels_list`, `social.content.artifact_get`, `social.content.preview`.
2. `social.content.artifact_create` saves a draft and publishes nothing.
3. `social.content.schedule` arms one delivery per channel and account. It does not call a provider. An unapproved draft returns `approval_required` true. Later dispatch refuses with `content_delivery_publish_authorization_required` until `social.content.approve` binds the current version.
4. `social.content.approve` stamps `approved_version`. It publishes nothing. It blocks an empty body, LinkedIn text above 3,000 characters, unresolved media, and unresolved mentions.
5. `social.content.delivery_publish` dry run returns `provider_payload` and `approved_publish_hash`. No provider call.
6. Live `social.content.delivery_publish` sends that hash. The post is public when the provider commits. `replayed` true means it was already published. `reconciliation_required` true means stop and read the delivery. Do not call `social.content.post_retry` until the delivery is `failed`.
7. A request to publish now still walks steps 2 to 6 when `social.posts.create` cannot carry the post. When the person already asked to publish that exact text, the dry run is the confirm step. Do not add a second question. If the server refuses and names an approval it needs (a governed workspace's reviewer pipeline, or consent for a paid or recurring run), get that approval from the person; never invent one.

A refused schedule stays on the same artifact. 409 `account_selection_required` names the accounts: bind one with `social.content.artifact_update`, then schedule again. 400 `social_account_binding_required` means no channel or no account. `provider_contract_not_found` means the LinkedIn schedule wrote no delivery. `channel_media_required` and `channel_media_kind_unsupported` mean fix the media, then schedule again. Do not open a second artifact. `social.content.approve` also refuses a stored unresolved media reference, so finish `social.content.media_uploaded` (and `runs_get` when that call queued) before approve.

## Creator posts

A creator program is not this queue. `social.ugc_programs.create` makes a program and leaves amplification off. `social.ugc_creators.invite` is link-only unless communication consent and the frozen preview token are both present. `social.ugc_posts.enroll` copies a creator draft in as unapproved and never publishes. `social.ugc_posts.approve` binds the current `enrollment_version`, and only the creator or a delegate the creator granted may approve. Do not schedule that enrollment with `social.content.schedule`.

## Schedule and publish

Scheduling is separate from drafting: schedule only when the user asks. Publishing is outward delivery. Follow the confirm-first ladder above.

Approval has separate axes: the user authorizes the intended outward effect, the current content version may need workspace approval, the provider or account must be ready, and paid or recurring work follows the paid-run consent rule. Review exact channels, accounts, time, copy and any spending ceiling. A discussion or editorial review alone does not authorize dispatch. A content change can invalidate version-bound approval; read current state and obtain any required new approval.

If a publish outcome is unknown, read the post and its delivery state with social.content.artifact_get and inspect the provider receipt and run evidence before any retry. A reconciling delivery is not proof of failure. Never republish blindly; if the provider outcome cannot be established, report the unresolved state and preserve the delivery for reconciliation.

Worked example: for a requested LinkedIn draft and scheduled slot, inspect the schemas, then save the supplied body and channel with social.content.artifact_create using {content: "Onboarding taught us to listen. What would you change?", channels: ["linkedin"]}. Use its returned artifact ID and revision, resolve the account and timezone, and call social.content.schedule with {artifact_id: returned ID, expected_revision: returned revision, scheduled_at: requested ISO timestamp}. Read social.content.artifact_get with {artifact_id: returned ID}; verify each delivery, scheduled_at, account and current approval state. If approval is required, follow the returned gate before dispatch; a scheduled result is not a published receipt.

When the user states a cadence, map every draft in the plan to a concrete slot and schedule all in one batch. If post count and slots conflict, ask before scheduling. Read back every scheduled post and report its date and time.

A scheduled long-form article is not a scheduled post: its publish time lives on a delivery attempt. Find the pending attempt with `social.content.article_deliveries_list`, then move it with `social.content.article_delivery_reschedule` or stop it with `social.content.article_delivery_cancel`.

Resolve one explicit IANA timezone. Convert requested local time once into one timestamp for every channel. Never reinterpret “morning” per channel.

Read the saved draft, schedule or delivery result before reporting it done. When you change one channel of a post, send only that channel's copy, then read the post back and compare every other channel with what it was before your change; if any channel lost or changed copy you did not mean to touch, restore it before you report. Never say another channel is unchanged without reading it back. Never claim a result from the request alone.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
