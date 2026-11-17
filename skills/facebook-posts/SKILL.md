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

Before drafting, check what has worked for this account on Facebook (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. `insufficient_data`, `not_recorded`, and `posts_without_metrics` are not zeros. Compare this Page only.

## Decide the idea

Return PASS, REPAIR, or REJECT. REPAIR names the failed gate and the proof still needed. Compare passing ideas in pairs. Do not score them from 1 to 10. A fully specified post does not stop for a vote.

1. A receipt the person or Company Context supplied.
2. A first-person claim the author lived.
3. Useful to people who already know this Page.
4. Specific: a period, a named event, or a before and after.
5. One idea. Do not invent a pillar.
6. Public-safe. A number needs a definition, a unit, a period, a source, and public-use clearance. A link must be one the person or the workspace supplied. Do not invent a destination.

Facebook does not borrow LinkedIn's "always a comment CTA" rule. A link post ends when the reason to care is clear and the URL sits after that opening. A text post can end on the point. The negation pivot at most once. No slogan closer. No "The catch?" transition.

## Voice and skim

The person's wording wins. Read `social.brand_voice_get`. If it is empty, `knowledge.brain.context.search` then `knowledge.brain.context.get`. Missing voice does not block an edit of supplied copy.

`social.content.artifact_list`, then `social.content.artifact_get` on the last two published posts for this Page. Do not repeat the same opening move. Skim line 1 and the line before the URL. Both must make sense with the link preview hidden.

## Save and read back

This skill writes the copy and can save the draft. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule.

When this skill saves, `social.content.artifact_create` once: `channels` `["facebook"]`, `status` `"draft"`, `content` the post, and the same text in `platform_payloads.facebook.message` when you set a Facebook payload. `format` is `feed` or `reel`. If social is already saving a multi-channel draft, return the Facebook text and do not create a second artifact. If no account is bound, omit `account_ids`, set `allow_unbound_draft` to true, and say so.

Read `social.content.artifact_get`. The stored message still has no URL in the opening. A 404 means the id is not in this workspace. An edit is `social.content.artifact_update` with `expected_revision` from `artifact.revision`. 409 `published_immutable` or 409 `stale_action` means stop and re-read.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The Facebook account is connected.
- Media is present when the post is a photo or video. A text-only post does not need media. A photo or video post does.
- The body is at or under 63,206 characters and still reads as conversation, not as a caption stuffed to the limit.

## The post

- Open with the point in plain speech. Short paragraphs. One idea.
- Do not pad toward 63,206. Most posts should be short enough to read without expanding.
- No em dashes, en dashes, or double hyphens.

## Media

A photo or video post carries images or video, up to 10. A text-only post needs none.
Attach media with the social skill's upload flow.

## Links

- The opening has no URL. The first lines must make sense with no link preview.
- Put the URL after the conversational opening, on its own line.
- Use only a link the user or the workspace supplied. Do not invent a destination.
- If the link is the point of the post, the sentence before it still says why a person should care.

## Editing

Change only what was asked. Moving a link into the first line is a mistake unless the user explicitly asked for that.

## Refused save and failed readback

The catalog has no Facebook privacy field. Do not invent one. `format` is `feed` or `reel` only.

Media and link failures, exact strings:

- "Facebook posts require text, a link, or media."
- "Facebook link posts require an HTTP(S) link and no attached media."
- "Facebook media requires an HTTP(S) URL."
- "Facebook photos must not exceed 10MB."
- "facebook publishing supports one video or up to 10 images."
- "Facebook Reels require one video."

`channel_media_too_many` and `channel_media_kind_unsupported` name the channel limit. A reel is `format` `reel` plus one video. Fix the same artifact with `social.content.artifact_update`. Do not create a second one.

A 404 from `social.content.artifact_get` means list with `social.content.artifact_list` first. 409 `published_immutable` or 409 `stale_action` means stop and re-read. If the stored message puts a URL in the opening, report the stored text and patch once with `expected_revision`.

## Check it works

Read the saved draft back. Confirm the account, media when the post needs it, the length, that the opening has no URL, that every number was supplied, and that nothing contains an em dash. If you saved, `social.content.artifact_get` matches.

## Worked example

Supplied link: `https://example.com/opener`. Supplied fact: the opener booked 4 meetings between 12 and 22 March. The person did not ask for a comment CTA.

Bad post:

```
https://example.com/opener
We 10xed replies. It's not the tool. It's the mindset.
```

Fixed post:

```
The opener named the missed reply, and it booked 4 meetings between 12 and 22 March.

https://example.com/opener
```

The bad post puts the URL first, invents "10xed", and uses a negation pivot with no receipt behind it. The fix leads with the supplied result. The link is on its own line after that sentence. No extra ask.

Save: `social.content.artifact_create` with that `content`, `channels` `["facebook"]`, `status` `"draft"`. Read back with `social.content.artifact_get` and confirm the first line is not a URL.

Paid and outward runs follow the paid-run consent rule.
