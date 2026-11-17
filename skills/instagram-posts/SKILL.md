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

Before drafting, check what has worked for this account on Instagram (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. `insufficient_data`, `not_recorded`, and `posts_without_metrics` are not zeros. Compare this account only.

## Decide the idea

Return PASS, REPAIR, or REJECT. REPAIR names the failed gate and the proof still needed. Compare passing ideas in pairs. Do not score them from 1 to 10. A fully specified caption does not stop for a vote.

1. A receipt the person or Company Context supplied.
2. A first-person claim the author lived. Someone else's result is not "we achieved".
3. Useful to the buyer in Company Context.
4. Specific: a period, a named event, a workflow, or a before and after.
5. Fits known pillars. Do not invent one. One post, one idea.
6. Public-safe. A number needs a definition, a unit, a period, a source, and public-use clearance. Remove an uncleared number and name it beside the caption.

Do not shorten a LinkedIn post and call it a caption. Rewrite the same evidence for a 125-character hook, line breaks, and one close.

## Voice, hook, skim

The person's wording wins. Read `social.brand_voice_get`. If it is empty, `knowledge.brain.context.search` then `knowledge.brain.context.get`. Missing voice does not block an edit of supplied copy. Never import another author's slang.

`social.content.artifact_list`, then `social.content.artifact_get` on the last two published posts for this account. The next hook is a different move: a claim, a scene, or a question. The negation pivot at most once. No "The catch?" transition. No slogan as the last line instead of the call to action.

Skim the caption as the first 125 characters, the first words of each beat, and the close. Skim a carousel as each slide's first line. If a slide's line does not state its beat, rewrite the line. If deleting a slide still tells the story, delete it.

## Save and read back

This skill writes the copy and can save the draft. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule.

When this skill saves, `social.content.artifact_create` once: `channels` `["instagram"]`, `status` `"draft"`, `content` the caption. Set `platform_payloads.instagram.format` to `feed`, `reel`, or `story`. A story is one item. A reel is one video. A feed carousel is 6 to 10 images, one idea per slide, slide 1 readable as a still. If social is already saving a multi-channel draft, return the caption and the format and do not create a second artifact. If no account is bound, omit `account_ids`, set `allow_unbound_draft` to true, and say so.

Publishing with no image or video returns `channel_media_required` and "Instagram needs at least one image or video." A script without media is not ready. Social attaches the file. This skill does not swap a missing video for a still.

Read `social.content.artifact_get`. `artifact.content` equals the caption. A 404 means the id is not in this workspace. An edit is `social.content.artifact_update` with `expected_revision` from `artifact.revision`. 409 `published_immutable` or 409 `stale_action` means stop and re-read.

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

## Media

Up to 10 images as a carousel, or 1 video. A GIF is converted to video before it posts. A Story is exactly 1 item. No documents.
Attach media with the social skill's upload flow.

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

## Refused save and failed readback

`channel_content_too_long` means the caption is over the registry limit. Shorten it and update the same artifact. Schedule without an image or video returns `channel_media_required` and "Instagram needs at least one image or video." Attach media through the social upload flow, then schedule that same artifact. Do not swap a missing video for a still, and do not create a second artifact.

A 404 from `social.content.artifact_get` means the id is not in this workspace. List with `social.content.artifact_list` first. 409 `published_immutable` or 409 `stale_action` means stop and re-read. If `artifact.content` differs from the caption you sent, report the stored caption and patch once with `expected_revision`.

## Check it works

Read the saved draft back. Confirm the limit, the 125-character hook, the hashtag count and position, the single call to action, the skim, and that nothing contains an em dash. If you saved, `social.content.artifact_get` matches.

## Worked example

Supplied: the person filmed a 20-second reel of the opener that booked 4 meetings between 12 and 22 March. They want one comment keyword, LINES. No other number, and no save-or-follow ask.

Bad caption:

```
We 10xed pipeline with a simple mindset shift. Comment LINES, save this, and check the link in bio. #growth #mindset #hustle #fyp #viral
```

Fixed caption:

```
The opener named the missed reply. It booked 4 meetings from 12 to 22 March.

Comment LINES and I will paste both lines.

#outbound
```

The bad caption invents "10xed", stacks three asks, and pads hashtags. The claim in the fix fits inside the first 125 characters. One ask. One honest tag. If the 4 meetings were missing, REPAIR and leave the number out. Carousel slide lines, if any, would each be one beat. A slide that only says "And that's the secret" gets cut.

Save: `social.content.artifact_create` with that caption, `channels` `["instagram"]`, `status` `"draft"`, `platform_payloads.instagram.format` `"reel"`. Read back with `social.content.artifact_get`.

Paid and outward runs follow the paid-run consent rule.
