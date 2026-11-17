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

Before drafting, check what has worked for this account on TikTok (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. `insufficient_data`, `not_recorded`, and `posts_without_metrics` are not zeros. Compare this account only.

## Decide the idea

Return PASS, REPAIR, or REJECT. REPAIR names the failed gate and the proof still needed. Compare passing ideas in pairs. Do not score them from 1 to 10. A fully specified script does not stop for a vote.

1. A receipt the person or Company Context supplied.
2. A first-person claim the author lived.
3. Useful to the buyer in Company Context.
4. Specific enough to say in one line: a period, a named event, or a before and after.
5. One idea. A second idea is a second video. Do not invent a pillar.
6. Public-safe. A number on screen needs a definition, a unit, a period, a source, and public-use clearance. Remove an uncleared number.

The spoken hook and the on-screen line are the same idea. Families to rotate: result, confession plus change, stakes, before and after. `social.content.artifact_list` then `social.content.artifact_get` on the last two published videos. Do not repeat that family's opening. No "hey guys", no logo sting, no context before the claim. The negation pivot at most once. The caption is not a transcript of the script.

## Voice

The person's wording wins. Read `social.brand_voice_get`. If it is empty, `knowledge.brain.context.search` then `knowledge.brain.context.get`. Missing voice does not block an edit of supplied copy. Never import another author's slang.

## Save and read back

This skill writes the script and the caption and can save the draft. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule.

When this skill saves, `social.content.artifact_create` once: `channels` `["tiktok"]`, `status` `"draft"`, `content` the caption. Put `privacy_level` under `platform_payloads.tiktok.specifics.tiktok`, chosen only from `creator_info_privacy_level_options` for that account. Do not guess. A disallowed value fails schedule with 422 `tiktok_privacy_option_invalid` and the message `TikTok does not allow privacy_level "<value>" for this account. Allowed values: <list>.` Use a value from that list. `tiktok_app_unaudited` means "TikTok only allows this app to post to private accounts until TikTok finishes its audit. Set the TikTok account to private, or wait for the audit." That is a schedule block for social, not a reason to invent a public privacy level.

`platform_payloads.tiktok.format` is `video` or `photo`. A photo title is held to 90 characters at publish. The caption stays at or under 2,200 characters. If social is already saving a multi-channel draft, return the script, caption, and privacy choice and do not create a second artifact. If no account is bound, omit `account_ids`, set `allow_unbound_draft` to true, and say the privacy options are still unknown.

Publishing with neither video nor photo returns `channel_media_required` and "TikTok needs a video or at least one photo." A script alone is not ready. Social attaches the file.

Read `social.content.artifact_get`. A 404 means the id is not in this workspace. An edit is `social.content.artifact_update` with `expected_revision` from `artifact.revision`. 409 `published_immutable` or 409 `stale_action` means stop and re-read.

## Before scheduling

Confirm all of these. If one is missing, say so and do not schedule.

- The TikTok account is connected.
- Video media is present. A script with no video is not ready.
- A privacy option is chosen: who can view the video. The stored field is privacy_level, nested under tiktok specifics. Do not leave it blank, and do not guess a value the account did not offer.
- The caption is at or under 2,200 characters and the hashtags are in range.

## Media

A post carries images or 1 video.
Attach media with the social skill's upload flow.

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

## Refused save and failed readback

Missing specifics fail with `tiktok_specifics_required` and "TikTok publish requires platform_payloads.tiktok.specifics.tiktok." Read `social.channels.creator_options_get` and set `privacy_level` to one returned option. 422 `tiktok_privacy_option_invalid` ("TikTok does not allow privacy_level ..." or "TikTok privacy_level is not one of the creator options.") means update the same artifact with a value from the allowed list in the error. Do not invent `PUBLIC_TO_EVERYONE`. `tiktok_app_unaudited` stays a schedule block. Do not invent a public level to get past it.

`channel_media_required` ("TikTok needs a video or at least one photo.") and "tiktok publishing requires one video URL." mean attach the file, then schedule the same artifact. A photo title over 90 characters returns "TikTok photo title must be 90 characters or fewer." "TikTok photo posts need a verified media domain in the TikTok developer console. Video posts upload directly." means switch that post to a direct video upload or stop. A `publish_id` prefixed `pending:` is not a live post. Read the delivery. Do not republish.

A 404 from `social.content.artifact_get` means list first. 409 `published_immutable` or 409 `stale_action` means stop and re-read. If the stored caption differs, report it and patch once with `expected_revision`.

## Check it works

Read the saved draft back. Confirm the account, the video or photo, the privacy option is one the account offered, the caption limit, the hashtag count, the on-screen line matches the first spoken line, and nothing contains an em dash. If you saved, `social.content.artifact_get` matches.

## Worked example

Supplied: a creator who will say, on camera, that naming the missed reply booked 4 meetings between 12 and 22 March. Privacy options for the account are only `PUBLIC_TO_EVERYONE`. No other number.

Bad script, first two seconds: "Hey guys, so today I want to talk about mindset." On-screen: "10x your pipeline". Caption pastes the whole script and ends with eight tags.

Fixed: first spoken line and on-screen line are "Named the missed reply." Next beat: "4 meetings, 12 to 22 March." Close: "Comment LINES." Caption, not a transcript:

```
Named the missed reply. 4 meetings from 12 to 22 March. Comment LINES and I will paste both openers.

#outbound
```

The bad opening wastes the first two seconds and invents 10x. `SELF_ONLY` is not in the allowed list, so it is not the privacy value. If the 4 meetings were missing, REPAIR and do not put a number on screen.

Save: `social.content.artifact_create` with the caption as `content`, `channels` `["tiktok"]`, `status` `"draft"`, `platform_payloads.tiktok.format` `"video"`, and `platform_payloads.tiktok.specifics.tiktok.privacy_level` `"PUBLIC_TO_EVERYONE"`. Read back with `social.content.artifact_get`.

Paid and outward runs follow the paid-run consent rule.
