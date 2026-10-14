---
name: x-posts
description: "Write and revise X posts and threads: the first line, single post versus thread, weighted length and Premium limits, thread structure, post types, replies and links, and phrases to avoid. Use it for the copy of any post going to X. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# X posts

## Boundary

- Owns: X post and thread copy.
- Post lifecycle, review, and publishing: [`social`].
- The `social` pack lists the tools and their exact inputs.

Write for a feed that discards most posts in under two seconds. Every line earns the next: the hook earns post two, post two earns post three.

## Single post or thread

Thread only when the idea has real sequential beats: an argument, a teardown, a multi-step how-to, a story with a turn. If it fits in one post, keep it one post. When the user asks for a thread, write a thread. Default thread length is 5 to 7 posts; 3 to 5 for a tight point; 10 or more only when every post adds something. Five dense posts beat nine padded ones.

A thread is saved as the full ordered list of posts, the first one included.

## The hook

The first 5 to 8 words decide whether anyone reads on. Lead with the claim, the number or the scene; never warm up ("so I was thinking") or hedge ("maybe", "kind of"). Match the pattern to the content:

- Bold claim: one flat sentence.
- Contrarian: name the popular belief and invert it with a mechanism.
- Stat: a real number, the person's own result or a sourced external one.
- Question: the one the reader wants answered, with stakes.
- Story: open mid-scene on a specific moment.
- Value promise: name what the reader walks away with, then deliver it.

Pick one emotional lever (status, us versus them, insider knowledge, tension, vulnerability, a quotable line) and commit. Never invent a number or fact.

## Thread structure

1. Hook (unnumbered). It must work as a great post even if nobody reads on, and it carries the whole promise of the thread.
2. Context (optional, threads of 7 or more): why this, why now, why from you.
3. Value posts, numbered inline on the same line: `2/`, `3/`, `4/`. One point, example or step each. Every post must survive as a standalone screenshot. If you can delete one and the thread still lands, delete it.
4. Close: a one-line takeaway or quotable punchline, then a specific ask or a declarative end. X posts do not need a CTA every time.

Alternate long and short posts to reset attention. Open a tension in the hook and resolve it in the close.

## Post types

- **One-liner**: one post, around 220 characters, the post is the content. Punch first, do not start with "I".
- **Question**: one specific question answerable from experience, stated directly, optionally one sentence of stakes.
- **Paragraph**: lead in the first 5 to 8 words, 1 to 3 short sentences of support, optional quotable landing line. Too long? Cut support, never split it.
- **List**: a real count and outcome in the promise, items with the useful phrase first. No padding to a round number.
- **Story**: specific scene, tension, the turn, one earned line of lesson. Never "and that's when I learned".
- **Thread**: the structure above, for arguments, teardowns and multi-part ideas.

## Length

Standard accounts get 280 weighted characters per post; Premium accounts get up to 25,000, though shorter still reads better. Weighted: a link of any length counts 23, emoji and CJK count 2. On a 280 limit aim for 240 to 260. The channel registry has the real limit for the account; read it, never assume.

## Voice and formatting

Write like a person: direct, sentences under about 20 words, varied openings. Line breaks mark the turn or the quotable line. No filler ("very", "really", "just", "basically"). No em or en dashes. Hashtags: none by default, at most one real community tag. Emoji: at most one per post. Put links in a reply or the last post, never inside the hook. Mention people only with context, and resolve every mention before scheduling.

## Never write

"🧵", "A thread:", "Here's what I learned:", "Hot take:", "Let me break this down", "In this thread I'll", "That's it for today", "End of thread", bare "Thoughts?" or "Agree or disagree?", and cliffhangers that the next post does not pay off.

## Editing

Change what was asked and keep the rest verbatim. If a thread post runs long, split it into another numbered post and renumber instead of shrinking the idea. Revise the existing draft instead of creating a near duplicate.

## Check it works

Read the saved draft back: the hook alone justifies post two, each post is under the account's limit, each thread post stands alone, every number traces to something real, and nothing contains an em dash.

Paid and outward runs follow the paid-run consent rule.
