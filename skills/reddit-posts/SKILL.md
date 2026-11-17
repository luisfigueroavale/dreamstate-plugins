---
name: reddit-posts
description: "Write and publish Reddit posts: subreddit choice and rules, the title, a value first body, disclosure, post types (success story, guide, resource list, problem solving, behind the scenes, question), and what gets posts removed. Use it for the copy of any Reddit draft. It does not cover scheduling or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Reddit posts

## Boundary

- Owns: Reddit title, body, and subreddit copy.
- Post lifecycle, review, and publishing: [`social`].
- The `social` pack lists the tools and their exact inputs.

This skill writes the title, the body, and the target subreddits. Social stores the draft. Do not claim the post is live on Reddit until `social.content.delivery_publish` returns a provider receipt. `social.content.artifact_create` accepts channel `reddit`. A connected Reddit account publishes through `social.content.schedule` and `social.content.delivery_publish`. `social.sender_profile_reddit_upsert` stores a label only and does not connect an account. Reddit punishes anything that reads like marketing, so write every draft as a real member of that community would.

Before drafting, check what has worked for this account on Reddit (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. `insufficient_data`, `not_recorded`, and `posts_without_metrics` are not zeros. Compare this account only. A thin sample is a hint, not a rule for which subreddit to pick.

## Pick the subreddit, then respect it

Match subreddits to where the topic and register fit, not to size: a founder sub reads differently from a practitioner or general discussion sub. Mirror the community's language without copying posts. If the sub's rules are not in Company Context or the brief, say that gap. Do not assume the sub allows a product mention, a link, or a missing flair. Karma minimums and self-promotion rules vary, and some subs ban self-promotion outright. With low or unknown karma, write pure value and no self-promotion.

## Decide the idea

Return PASS, REPAIR, or REJECT. REPAIR names the failed gate and the proof still needed. Compare passing angles in pairs. Do not score them from 1 to 10. A fully specified story does not stop for a vote.

1. A receipt the person or Company Context supplied.
2. A first-person claim the author lived. A third party's result is not "we achieved".
3. Useful to that subreddit, not only to a buyer.
4. Specific: a period, a named tool, a workflow, or a before and after.
5. Fits the sub's register. Do not invent a community norm.
6. Public-safe. A number needs a definition, a unit, a period, a source, and public-use clearance. Do not round it. An uncleared number comes out of the post.

Own product is at most a tenth of the post, disclosed in plain language ("I built a tool for this"). No links in the body. The negation pivot at most once. No slogan closer. A TL;DR stays on posts over 500 characters, because Reddit readers use it. That exception is Reddit only.

Voice: the person's wording wins. Read `social.brand_voice_get`. If it is empty, `knowledge.brain.context.search` then `knowledge.brain.context.get`. Missing voice does not block an edit of supplied copy.

`social.content.artifact_list`, then `social.content.artifact_get` on the last two Reddit drafts for this account. Do not reuse the same title pattern. Skim the title, each `##` header, and the last line. The title must not repeat as the first body line. If the body focus changes, change the title.

## The title carries the hook

Front-load the outcome, claim or question, never setup. Maximum 300 characters, aim for 60 to 120. Patterns that work, always with real numbers the person gave you: a big specific number, a story hook, a lesson count, a direct ask ("Roast my landing page"), a problem and its fix. No clickbait, hashtags or emoji. Do not repeat the title as the first line of the body. If the body's focus changes, change the title.

## Body: hook, value, soft close

- **Hook** (2 to 3 sentences): the most compelling point, and a specific number only when one was supplied.
- **Value** (the bulk): deliver everything the title promised, in full, with specifics. Give the whole playbook, not a teaser.
- **Soft close**: invite the conversation. Mention the person's own product only if it is genuinely relevant.

## Value first, never a pitch

At most a tenth of the post may touch the person's own product, and only naturally. Lead with the problem, never the product name. When the product is mentioned, disclose it plainly ("I built a tool for this"); never pose as a neutral recommender. No links in the body: describe the tool and offer details in the comments.

## Voice

Contractions always. Mix short and long sentences, and never start three sentences the same way. Include one specific, unrounded detail only someone who did the thing would know ($4,217, the tool that failed, the moment of doubt). No em dashes, en dashes or double hyphens. No emoji or hashtags unless asked.

## Formatting

Reddit markdown: bold for key points, `##` headers to break up long posts, numbered lists for steps, bullets for items. Paragraphs of 2 to 3 sentences. Add a TL;DR to anything over 500 characters.

## Post types

Follow the format the person named. Otherwise pick from what they have:

- **Success story** (a real milestone): open on the struggle or turning point, the win lives in the title. What failed first, the tactics with trade-offs, unrounded before and after, what still has not improved, 3 to 5 lessons, a question.
- **Guide** (a repeatable process they have run): the outcome and the common mistake, numbered steps each doable without more research and naming the exact tool, real pitfalls, an offer to help in the comments.
- **Resource list** (tools they actually used): selection criteria and how much they tested, items under headers with use, cost, one pro, one con, and who it is not for, plus 2 to 3 honest rejections.
- **Problem solving** (a named problem and a replicable fix): the problem and its cost, what did not work, the approach that worked with a timeline, results, the limits of their context.
- **Behind the scenes** (the building process): current state, real numbers including unimpressive ones, the real setbacks, decisions and why, a real question.
- **Question or discussion** (they want input): their own experience first and at length, why they are asking, one specific open question that is not searchable, a promise to reply.

## What gets posts removed or downvoted

Round or invented numbers, a thin wrapper around a pitch, missing flair or disclosure, humble bragging, generic advice with no numbers, and phrases like "delve into", "unlock", "leverage", "game-changer", "in today's world". Leave one specific question the author can answer. Replying in the thread is not this skill.

## Editing a draft

Keep untouched sections and markdown exactly. Edit at the phrase level instead of rewriting. Shortening cuts content and keeps the structure. Never invent facts or numbers.

## Refused save and failed readback

`reddit_title_required` ("Reddit requires platform_payloads.reddit.reddit_title.") and `reddit_subreddits_required` ("Reddit requires at least one platform_payloads.reddit.subreddits entry.") mean put those fields on the same draft and update it. Do not drop them to get a save.

A 404 from `social.content.artifact_get` means the id is not in this workspace. Call `social.content.artifact_list` before a second create. 409 `published_immutable` or 409 `stale_action` means stop and re-read. If the stored title or body differs, report the stored text and patch once with `expected_revision`.

## Check it works

Read the saved draft back: the title is under 300 characters and matches the body, subreddits are set, there are no links or em dashes, self-promotion is disclosed and small, and every number was supplied.

## Save and read back

When this skill saves, call `social.content.artifact_create` once with `channels` `["reddit"]`, `status` `"draft"`, `content` the body, and `platform_payloads.reddit.reddit_title` plus `platform_payloads.reddit.subreddits`. A publish-time check without them returns `reddit_title_required` ("Reddit requires platform_payloads.reddit.reddit_title.") or `reddit_subreddits_required` ("Reddit requires at least one platform_payloads.reddit.subreddits entry."). Put them on the draft now. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule. If social is already saving a multi-channel draft, return the title, body, and subreddits and do not create a second artifact.

Read `social.content.artifact_get`. The stored title matches the body. A 404 means the id is not in this workspace. An edit is `social.content.artifact_update` with `expected_revision` from `artifact.revision`. 409 `published_immutable` or 409 `stale_action` means stop and re-read.

## Worked example

Supplied: in a practitioner sub, the person changed an opener and booked 4 meetings between 12 and 22 March. They built the worksheet themselves. The sub's rules are not in Company Context. No other number exists.

Bad title: `I 10xed pipeline with my tool`

The bad body opens on the product name, invents 10x, and never says the person built the tool.

Fixed title: `Opener change booked 4 meetings (12 to 22 March). First version failed.`

The fixed body opens on the failed first line, then the missed-reply line, then the 4 meetings with those dates. The tool is one disclosed sentence: "I built the worksheet I used." There is no link. Beside the draft, say the sub's self-promotion rule is unknown, so that sentence stays one line and can be cut. The bad draft is REPAIR on the invented 10x. A body that is only the pitch is REJECT. The fixed draft is PASS only because the 4 meetings were supplied.

Save: `social.content.artifact_create` with the body as `content`, `channels` `["reddit"]`, `status` `"draft"`, `platform_payloads.reddit.reddit_title` set to the fixed title, and `platform_payloads.reddit.subreddits` set to the named sub. Read back with `social.content.artifact_get`.

Paid and outward runs follow the paid-run consent rule.
