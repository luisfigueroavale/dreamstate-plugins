---
name: tiktok-posts
description: "Write and revise TikTok video scripts and captions: the first two seconds, on-screen text, a caption up to 2,200 characters, and 3 to 5 hashtags. Use it for the copy of any post going to TikTok. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# TikTok posts

## Boundary

- Owns: TikTok video script and caption copy.
- The `social` pack lists the tools and their exact inputs.

Post lifecycle, review, and publishing stay with the social skill. Voice comes from the account's past posts and the workspace brand voice.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The TikTok account is connected.
- Video media is present. A script with no video is not ready.
- A privacy option is chosen: who can view the video. The stored field is privacy_level, nested under tiktok specifics. Do not leave it blank, and do not guess a value the account did not offer.
- The caption is at or under 2,200 characters and the hashtags are in range.

## The first 2 seconds

Open on the point. The first words and the first on-screen line are the same idea. No "hey guys", no logo sting, no context before the claim.

## Script

- Hook, then the beats, then one close.
- On-screen text is a short line, not a paragraph.
- One idea. If the script needs a second idea, it is a second video.

## Caption

- Hard limit: 2,200 characters.
- 3 to 5 relevant hashtags at the end. Do not pad. If fewer than 3 are honest, use only those.
- The caption adds a reason to watch or to reply. It does not paste the script.
- No em dashes, en dashes, or double hyphens.

## Editing

Change only what was asked. A new hook does not license a new caption or a new privacy choice.

## Check it works

Read the saved draft back. Confirm the account, the video, the privacy option, the caption limit, the hashtag count, and that nothing contains an em dash.

Paid and outward runs follow the paid-run consent rule.
