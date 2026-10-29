---
name: instagram-posts
description: "Write and revise Instagram captions, carousel outlines, and Reels scripts: the first 125 characters, line breaks, 3 to 8 hashtags, and a comment, save, or link-in-bio close. Use it for the copy of any post going to Instagram. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Instagram posts

## Boundary

- Owns: Instagram caption, carousel outline, and Reels script copy.
- The `social` pack lists the tools and their exact inputs.

Post lifecycle, review, and publishing stay with the social skill. Voice comes from the account's past posts and the workspace brand voice. Structure comes from the mode below.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The Instagram account is connected.
- Media is present: one image or video for a feed post or Reel, and one image per carousel slide.
- The caption is at or under 2,200 characters, the first 125 characters are a complete hook, hashtags sit at the end, and the close is a real CTA.

## The first 125 characters

The grid and the feed cut the caption after about 125 characters. That window is the hook: a specific claim, scene, or question. It must make sense with no "more". Never invent a number to force the hook.

## Caption

- Line breaks carry the rhythm. A blank line between beats.
- 3 to 8 relevant hashtags, on their own lines at the end. Skip a tag you cannot justify. Fewer honest tags beat padded ones.
- One CTA: a comment keyword ("comment GUIDE"), a save, or the link in bio. Not all three unless asked.
- No em dashes, en dashes, or double hyphens.

## Carousel

- Slide 1 is the hook, within 125 characters, and it must work as a still image with almost no caption.
- 6 to 10 slides. One idea per slide. If a slide can be deleted and the sequence still lands, delete it.
- The last slide is the CTA, the same ask as the caption.

## Reels

- The hook is in the first 2 seconds, spoken or on screen.
- On-screen text is shorter than the spoken line and matches it.
- The caption still follows the caption rules. It is not a transcript dump.

## Editing

Change only what was asked. A new hook does not license a new CTA or a new hashtag block.

## Check it works

Read the saved draft back. Confirm the limit, the 125-character hook, the hashtag count and position, the CTA, and that nothing contains an em dash.

Paid and outward runs follow the paid-run consent rule.
