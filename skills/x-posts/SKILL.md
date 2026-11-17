---
name: x-posts
description: "Write and revise X posts and threads: the first line, single post versus thread, weighted length and Premium limits, thread structure, post types, replies and links, and phrases to avoid. Use it for the copy of any post going to X. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# X posts

## Boundary

- Owns: X post and thread copy.
- Post lifecycle, review, and publishing: [`social`].
- The `social` pack lists the tools and their exact inputs.

Write for a feed that discards most posts in under two seconds. Every line earns the next: the hook earns post two, post two earns post three.

Before drafting, check what has worked for this account on X (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. Rows may include `insufficient_data`. The result may include `not_recorded` and `posts_without_metrics`. Those are not zeros. Compare this account's original posts only. A reply that starts with `@` is not a format test. Do not treat a LinkedIn winner as an X result.

## Decide the idea

Return PASS, REPAIR, or REJECT. REPAIR names the failed gate and the proof still needed. REJECT means the idea cannot be made honest. Compare ideas that pass, in pairs. Do not score them from 1 to 10. Offer 3 to 5 unless the brief already chose the angle. Do not stop a fully specified draft for a vote.

1. A receipt the person or Company Context supplied.
2. The author can stand behind a first-person claim. Another company's result is not "we achieved".
3. Useful to the buyer named in Company Context.
4. Specific: a period, a named event, a workflow, a company, or a before and after.
5. Fits pillars when Company Context has them. Do not invent a pillar. One post, one claim.
6. Public-safe: no invented metric, and no customer detail that is not cleared for public use.

A number needs a definition, a unit, a period, a source, and a public-use status. Do not round up. Remove an uncleared number from the copy and name it beside the draft.

Shortening a LinkedIn draft is not an X post. Rewrite from the same evidence: one claim, one receipt, one payoff for the reader. A thread only when the next posts add steps or evidence. Do not split a short thought to hit a post count. The opener must be a useful post even if nobody reads on.

## Voice, hook family, skim

The person's current wording wins. Read `social.brand_voice_get`. If X examples are missing, LinkedIn voice is provisional background: say so. If voice is empty, `knowledge.brain.context.search` then `knowledge.brain.context.get`. Missing voice does not block an edit of supplied copy. Never copy another author's slang or numbers.

`social.content.artifact_list`, then `social.content.artifact_get` on the last two published original posts for this account. Name the hook pattern already used (bold claim, contrarian, stat, question, story, value promise) and pick a different one. The negation pivot ("It is not X. It is Y.") at most once. No transition questions ("The catch?"). No aphorism as the only close when the hook promised steps.

Skim a thread as post 1, the first words of each later post, and the last post. If that skeleton drops the promise, cut or reorder. Do not pad.

## Save and read back

This skill writes the copy and can save the draft. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule.

When this skill saves, call `social.content.artifact_create` once with `channels` `["x"]`, `status` `"draft"`, and `content` set to the first post. A thread also sets `platform_payloads.x.thread_data` to the full ordered list, first post included. The publisher posts `thread_data` and ignores `content`, so those two must still match on the first post. Per-post media goes in `platform_payloads.x.thread_media`, parallel to `thread_data`, not in top-level `media`. If the save says "For an X thread with per-post media, put media in thread_media, not in media.", move the media and update once.

A poll is `platform_payloads.x.twitter_poll`: 2 to 4 `options`, each at most 25 characters, and `duration_minutes` from 5 to 10080. A poll cannot carry media. The refusal is "Provider polls and quote posts cannot be combined with media." A quote (`twitter_quote_tweet_id`) can carry media and cannot carry a poll. Alt text sits on the media item, describes what is in the image, and stays at or under 1000 characters. Over that, the refusal is "X alt text must be 1000 characters or fewer."

If the save returns `channel_content_too_long`, the message names the limit and, for a thread, the post number. Split that post. Do not trust a raw JavaScript string length. Weighted length is the registry rule already in this skill.

Read `social.content.artifact_get`. For a thread, `platform_payloads.x.thread_data` is the full order you wrote. A 404 means the id is not in this workspace. An edit uses `social.content.artifact_update` with `expected_revision` from `artifact.revision`. 409 `published_immutable` or 409 `stale_action` means stop and re-read. Do not open a second artifact. If social is already saving a multi-channel draft, return the X payload and do not create another artifact. If no account is bound, omit `account_ids`, set `allow_unbound_draft` to true, and say the account is missing.

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

## Media

Up to 4 images, or 1 GIF, or 1 video. No documents.
Attach media with the social skill's upload flow.

## Never write

"🧵", "A thread:", "Here's what I learned:", "Hot take:", "Let me break this down", "In this thread I'll", "That's it for today", "End of thread", bare "Thoughts?" or "Agree or disagree?", and cliffhangers that the next post does not pay off.

## Editing

Change what was asked and keep the rest verbatim. If a thread post runs long, split it into another numbered post and renumber instead of shrinking the idea. Revise the existing draft instead of creating a near duplicate.

## Check it works

Read the saved draft back: the hook alone justifies post two, each post is under the account's limit, each thread post stands alone, every number traces to something real, the hook family differs from the last two posts, and nothing contains an em dash. If you saved, `social.content.artifact_get` matches the text you sent.

## Worked example

Supplied fact: the same opener booked 4 meetings between 12 and 22 March when it named the missed reply instead of the product. No other number exists. The person asked to "turn the LinkedIn post into X" by shortening it.

Bad draft (a shortened LinkedIn post):

```
It's not the copy. It's the system. Distribution is the moat. We booked 4 meetings. Thoughts?
```

Fixed post:

```
Named the missed reply instead of the product.

That opener booked 4 meetings from 12 to 22 March.
```

The bad draft keeps the LinkedIn slogan, the negation pivot, and a stock question, and it would still be one LinkedIn paragraph with the line breaks removed. The fixed post is one claim and one receipt. It is under 280 weighted characters. It does not need a thread. If the dates had not been supplied, the verdict is REPAIR and the number stays out.

Save: `social.content.artifact_create` with `content` set to the fixed post, `channels` `["x"]`, `status` `"draft"`. Read back with `social.content.artifact_get`. Do not send `thread_data` for this single post.

## Poll example

The person asked for a one-day poll, two options, no image. Options stay at or under 25 characters.

```json
{
  "content": "Which opener should we keep?",
  "channels": ["x"],
  "status": "draft",
  "platform_payloads": {
    "x": {
      "twitter_poll": {
        "options": ["Missed reply", "Product name"],
        "duration_minutes": 1440
      }
    }
  }
}
```

`social.content.artifact_get` must show both options and `duration_minutes` 1440, and no media. If create returns "Provider polls and quote posts cannot be combined with media.", remove the media and save once. Do not convert the poll into `twitter_quote_tweet_id`.

## Refused save and failed readback

`channel_content_too_long` names the limit and, for a thread, the post number. Split that post and update the same artifact. A 404 on `social.content.artifact_get` means list with `social.content.artifact_list` and do not create a duplicate. 409 `published_immutable` or 409 `stale_action` means stop and re-read. If the stored `thread_data` or poll options differ from what you sent, report the stored payload and patch once with `expected_revision`.

Paid and outward runs follow the paid-run consent rule.
