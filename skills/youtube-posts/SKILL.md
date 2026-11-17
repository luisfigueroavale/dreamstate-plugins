---
name: youtube-posts
description: "Write and revise YouTube titles, descriptions, and tags: a title of at most 100 characters, a description with chapters, and a tag list. Use it for the copy of any video going to YouTube. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# YouTube posts

## Boundary

- Owns: YouTube title, description, chapters, and tags.
- The `social` pack lists the tools and their exact inputs.

Post lifecycle, review, and publishing stay with the social skill. Voice comes from the channel's past videos and the workspace brand voice.

Before drafting, check what has worked for this account on YouTube (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. `insufficient_data`, `not_recorded`, and `posts_without_metrics` are not zeros. Compare this channel only.

## Decide the title

The title is the hook. It must still be true if it is the only text on the thumbnail. Return PASS, REPAIR, or REJECT for the claim in the title. REPAIR names the missing proof. Do not score title options from 1 to 10. Offer 3 titles only when the person asked for options. A full video gets one title.

1. The claim is a receipt the person, the video, or Company Context supplied.
2. A first-person result is the author's. Do not retitle someone else's number as "we".
3. A viewer who matches the buyer can tell what they will get.
4. Specific: a real number, a contrast, or a named step. Never an invented number.
5. One promise. Do not invent a series pillar to fill the title.
6. Public-safe. A number needs a definition, a unit, a period, a source, and public-use clearance.

`social.content.artifact_list`, then `social.content.artifact_get` on the last two published videos. Do not repeat that title pattern. Read `social.brand_voice_get`. If it is empty, `knowledge.brain.context.search` then `knowledge.brain.context.get`. The person's wording wins. Chapters use only timestamps you can justify from the video or the brief. Do not invent a chapter to look complete.

Skim the title, the first description sentence, and the first chapter label. Together they must state the same promise. The negation pivot at most once in the description. No slogan in place of the first sentence.

## Save and read back

This skill writes the title, description, chapters, and tags, and can save the draft. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule.

When this skill saves, `social.content.artifact_create` once: `channels` `["youtube"]`, `status` `"draft"`, `content` the description. Put the package in `platform_payloads.youtube.snippet`: `title` at most 100 characters, `description` at most 5,000 characters including the `CHAPTERS:` block, `tags` as a list of search phrases whose joined text stays under 500 characters. Set `platform_payloads.youtube.status.privacyStatus` only when the request names `public`, `unlisted`, or `private`. Any other value fails the schema. If the request does not name one, leave it unset: the publisher then sends `private`. A `publishAt` on that status forces `private` for the upload. Do not set `public` to fill the gap. If social is already saving a multi-channel draft, return the snippet and do not create a second artifact. If no account is bound, omit `account_ids`, set `allow_unbound_draft` to true, and say so.

Publishing without exactly one video returns `channel_media_required` and "YouTube needs exactly one video." A title alone is not ready. Social attaches the file.

Read `social.content.artifact_get`. `snippet.title` matches the title you wrote. A 404 means the id is not in this workspace. An edit is `social.content.artifact_update` with `expected_revision` from `artifact.revision`. 409 `published_immutable` or 409 `stale_action` means stop and re-read.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The YouTube account is connected.
- Video media is present. A title with no video is not ready.
- The title is at or under 100 characters, the description has a chapters block, and the tags are present.

## Media

Exactly 1 video per post.
Attach it with the social skill's upload flow.

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

## Refused save and failed readback

`channel_media_required` with "YouTube needs exactly one video." means a title is not ready. Attach one video through the social upload flow, then schedule the same artifact. `channel_media_kind_unsupported` means the item is not video. Do not create a second artifact.

An unknown YouTube upload returns resume `youtube_resumable`. Read the delivery on `social.content.artifact_get` before another publish. Do not start a second upload while the delivery is reconciling.

A 404 from `social.content.artifact_get` means list with `social.content.artifact_list` and do not duplicate the draft. 409 `published_immutable` or 409 `stale_action` means stop and re-read. If `snippet.title` differs, report the stored title and patch once with `expected_revision`.

## Check it works

Read the saved draft back. Confirm the account, the video, the title length, the chapters, the tags, that the title does not invent a number, and that nothing contains an em dash. If you saved, `social.content.artifact_get` shows the same `snippet.title`.

No em dashes, en dashes, or double hyphens in the title, description, or tags.

## Worked example

Supplied: the video shows the opener change that booked 4 meetings between 12 and 22 March. The brief has two timestamps: 0:00 the old line, 1:10 the missed-reply line. No privacy choice. The last two titles were both "How to X" questions.

Bad title: `How I 10xed Pipeline in 7 Days`

Fixed title: `4 meetings from the missed-reply opener`

Fixed description opening:

```
The old line named the product. The new line named the missed reply, and it booked 4 meetings between 12 and 22 March.

CHAPTERS:
0:00 The old line
1:10 The missed reply
```

The bad title invents 10x and 7 days, and it repeats the question pattern of the last two videos. The fix uses the supplied count, stays under 100 characters, and leaves the dates in the description. It only includes the two chapters the brief justified. If the meeting count had not been supplied, the title would not contain a number and the verdict would be REPAIR.

Save: `social.content.artifact_create` with `channels` `["youtube"]`, `status` `"draft"`, `content` the description, and `platform_payloads.youtube.snippet.title` the fixed title. Do not set `privacyStatus`. Read back with `social.content.artifact_get`.

Paid and outward runs follow the paid-run consent rule.
