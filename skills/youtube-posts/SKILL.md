---
name: youtube-posts
description: "Write and revise YouTube titles, descriptions, and tags: a title of at most 100 characters, a description with chapters, and a tag list. Use it for the copy of any video going to YouTube. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, a live external send above the server's recipient threshold, or a run that turns on a recurring trigger, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# YouTube posts

## Boundary

- Owns: YouTube title, description, chapters, and tags.
- The `social` pack lists the tools and their exact inputs.

Post lifecycle, review, and publishing stay with the social skill. Voice comes from the channel's past videos and the workspace brand voice.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The YouTube account is connected.
- Video media is present. A title with no video is not ready.
- The title is at or under 100 characters, the description has a chapters block, and the tags are present.

## Title

At most 100 characters. Package it the way a video that outperforms its niche would: a specific claim, a real number, or a contrast. The title must still be true if it is the only text on the thumbnail. Never invent a number, name, or result.

## Description

- Say what the viewer gets, in the channel's voice, before any links.
- A `CHAPTERS:` block. The first line is `0:00` plus the opening label. Later lines are timestamps you can justify from the video or the brief.
- Description stays at or under 5,000 characters.

## Tags

One list, comma-separated, under 500 characters. Tags are search phrases for this video, not a hashtag dump in the title.

## Editing

Change only the part that was asked. A sharper title does not license a new description.

## Check it works

Read the saved draft back. Confirm the account, the video, the title length, the chapters, the tags, and that nothing contains an em dash.

No em dashes, en dashes, or double hyphens in the title, description, or tags.

Paid and outward runs follow the paid-run consent rule.
