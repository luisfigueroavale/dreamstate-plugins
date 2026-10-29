---
name: facebook-posts
description: "Write and revise Facebook posts: conversational copy up to 63,206 characters, and where a link sits so the opening still reads as a person. Use it for the copy of any post going to Facebook. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Facebook posts

## Boundary

- Owns: Facebook post copy, including posts that carry a link.
- The `social` pack lists the tools and their exact inputs.

Post lifecycle, review, and publishing stay with the social skill. Voice comes from the Page's past posts and the workspace brand voice. Write like a person talking to people who already know them.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The Facebook account is connected.
- Media is present when the post is a photo or video. A text-only post does not need media. A photo or video post does.
- The body is at or under 63,206 characters and still reads as conversation, not as a caption stuffed to the limit.

## The post

- Open with the point in plain speech. Short paragraphs. One idea.
- Do not pad toward 63,206. Most posts should be short enough to read without expanding.
- No em dashes, en dashes, or double hyphens.

## Links

- The opening has no URL. The first lines must make sense with no link preview.
- Put the URL after the conversational opening, on its own line.
- Use only a link the user or the workspace supplied. Do not invent a destination.
- If the link is the point of the post, the sentence before it still says why a person should care.

## Editing

Change only what was asked. Moving a link into the first line is a mistake unless the user explicitly asked for that.

## Check it works

Read the saved draft back. Confirm the account, media when the post needs it, the length, that the opening has no URL, and that nothing contains an em dash.

Paid and outward runs follow the paid-run consent rule.
