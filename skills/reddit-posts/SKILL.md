---
name: reddit-posts
description: "Draft Reddit posts the user posts themselves: subreddit choice and rules, the title, a value first body, disclosure, post types (success story, guide, resource list, problem solving, behind the scenes, question), and what gets posts removed. Use it for the copy of any Reddit draft. It does not cover scheduling or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. A larger paid run also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# Reddit posts

## Boundary

- Owns: Reddit post and thread copy the user posts themselves.
- Post lifecycle, review, and publishing: [`social`].
- The `social` pack lists the tools and their exact inputs.

Reddit is draft only: you write the title, the body and the target subreddits, and the user posts it themselves. Reddit punishes anything that reads like marketing, so write every draft as a real member of that community would.

## Pick the subreddit, then respect it

Match subreddits to where the topic and register fit, not to size: a founder sub reads differently from a practitioner or general discussion sub. Mirror the community's language without copying posts. Tell the user to check each sub's rules and recent top posts before posting: karma minimums, required flair and self-promotion rules vary, and some subs ban self-promotion outright. With low or unknown karma, write pure value and no self-promotion.

## The title carries the hook

Front-load the outcome, claim or question, never setup. Maximum 300 characters, aim for 60 to 120. Patterns that work, always with real numbers the person gave you: a big specific number, a story hook, a lesson count, a direct ask ("Roast my landing page"), a problem and its fix. No clickbait, hashtags or emoji. Do not repeat the title as the first line of the body. If the body's focus changes, change the title.

## Body: hook, value, soft close

- **Hook** (2 to 3 sentences): the most compelling point and a specific number.
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

Round or invented numbers, a thin wrapper around a pitch, missing flair or disclosure, humble bragging, generic advice with no numbers, and phrases like "delve into", "unlock", "leverage", "game-changer", "in today's world". Remind the user that the post works only if they stay and reply to every comment for the first few hours, critical ones included.

## Editing a draft

Keep untouched sections and markdown exactly. Edit at the phrase level instead of rewriting. Shortening cuts content and keeps the structure. Never invent facts or numbers.

## Check it works

Read the saved draft back: the title is under 300 characters and matches the body, subreddits are set, there are no links or em dashes, and self-promotion is disclosed and small.

Paid and outward runs follow the paid-run consent rule.
