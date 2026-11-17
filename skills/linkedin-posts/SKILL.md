---
name: linkedin-posts
description: "Write and revise LinkedIn posts that earn the click and the reply: the 210 character hook, structure and formatting, length by post type (story, list, case study, belief disruption, short, paragraph), specificity, the CTA close, and phrases to avoid. Use it for the copy of any post going to LinkedIn. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# LinkedIn posts

## Boundary

- Owns: LinkedIn post copy.
- Post lifecycle, review, and publishing: [`social`].
- The `social` pack lists the tools and their exact inputs.

Write every LinkedIn post so it survives two moments: the feed preview and the close. Everything between them keeps the promise the first three lines made. Voice comes from the account's past posts and brand voice; structure comes from the post type below.

Before drafting, inspect collected inspiration examples when a user names a genre, topic, author, or reference style. Use only examples available to the active workspace, treat their text as structure evidence rather than copy, and say when the library has no matching classified genre. Browse curated templates when the user wants a proven structure, then adapt the selected template to the workspace evidence and voice. A template is guidance, never proof for a factual claim and never permission to copy an example.

Before drafting, check what has worked for this account on LinkedIn (format, length, hook, posting hour) with `social.content.performance_query` (group by post format, published hour or weekday, and read engagement rate, which is likes, comments, reposts and saves over impressions for the same posts at the same age and is empty when impressions are not recorded), and treat a small sample as a hint. Rows may include `insufficient_data`. The result may include `not_recorded` and `posts_without_metrics`. Those are not zeros. Compare this account only. Do not import another channel's winner.

## Decide the idea

Return PASS, REPAIR, or REJECT. REPAIR names the failed gate and the proof still needed. REJECT means the idea cannot be made honest. Compare ideas that pass, in pairs. Do not score them from 1 to 10. Offer 3 to 5 unless the brief already chose the angle. A fully specified draft does not stop for a vote.

1. A receipt the person or Company Context supplied: a result, a call, an event, a metric, or attributed research.
2. The author can stand behind a first-person claim. Another company's result is not "we achieved".
3. Useful to the buyer named in Company Context.
4. Specific: a period, a named event, a workflow, a company, or a before and after.
5. Fits pillars and positioning when Company Context has them. Do not invent a pillar to fill a slot. One post has one job: attention, a method, or the offer.
6. Public-safe: no invented metric, and no customer detail that is not cleared for public use.

A number needs a definition, a unit, a period, a source, and a public-use status. Do not round up. Do not add unrelated metrics. Remove an uncleared number from the copy and name it beside the draft.

## Voice, hook family, skim

The person's current wording wins over these house rules. Read `social.brand_voice_get`. If voice is empty, `knowledge.brain.context.search` for voice and proof, then `knowledge.brain.context.get` on the page you will use. Missing voice does not block an edit of copy the person already supplied. Label that draft provisional. Never copy another author's slang, dialect, or numbers.

Before the hook, call `social.content.artifact_list` (newest updated first) and `social.content.artifact_get` on the last two published posts for this account. Name each hook family and use a different one. That rotation is mandatory: two result hooks in a row means this draft is not a result hook. Families: result plus a simple method, test or comparison, big claim, news plus result, confession plus change, stakes, before and after, receipt as the promise. `social.post_format_hooks_list` returns openings already analysed for one format. An empty list means that format has no analysis, not that the family is banned. `social.content.generate_hooks` stores candidates on a batch and does not write the artifact. `social.content.chat.hook_choice_apply` only marks the chosen candidate. `social.content.regenerate_hooks` overwrites that same batch and drops the previous candidates. Write the post yourself from the family you picked. `social.content.generate` spends credits and persists a draft. Use it only when asked, under the paid-run consent rule. Do not open with "Everyone is doing X. Almost nobody says Y." Line 1 does not name the product unless the person asked, or this account's own `social.content.performance_query` shows that product hooks win. A repeated receipt moves to line 2. A fading post gets a new mechanism, not a bigger number.

Skim test, before you show the draft: keep only line 1, the first words of each block, and the last line. If that skeleton still carries the argument, the structure holds. If it reads as fragments, pick another post type. Editing sentences will not fix it. A list head must stand alone ("1. Spec writing", not "1. The first thing").

On top of the phrase ban: the negation pivot ("It is not X. It is Y.") at most once, and only when the contrast is the story. No aphoristic closer that restates the topic as a slogan. No transition questions ("The result?", "The catch?", "Why does this matter?"). No throat-clearing (Furthermore, Moreover, Additionally, Ultimately).

## Save and read back

This skill writes the copy and can save the draft. Scheduling and publishing stay with social, which calls `social.content.schedule` and `social.content.delivery_publish` when the person asked to publish. `social.content.generate` spends credits. Use it only when the person asked for the paid generator, and then follow the paid-run consent rule. The default is to write the copy here.

When this skill is asked to save, call `social.content.artifact_create` once. When social is already saving one multi-channel draft, return the exact LinkedIn text and do not create a second artifact. Create params: `content` is the exact draft, `channels` is `["linkedin"]`, `status` is `"draft"`, `cta_policy` is `"refuse"`. Include `account_ids` when the account is already chosen. If no account is bound, omit `account_ids`, set `allow_unbound_draft` to true, and say the account is missing.

`cta_policy` `"append"` is the server default and the wrong choice here. A close that fails is then stored with "What would you change?" added. `"refuse"` returns 422 `linkedin_cta_required`. The message starts with "LinkedIn drafts must end with a call to action." Fix the close and create again. Do not flip the policy to append to get past the refusal.

A LinkedIn repost (`platform_payloads.linkedin.linkedin_repost_of`) does not get a call to action. Leave the commentary without one. LinkedIn polls cannot be published yet, so do not set `linkedin_poll`.

Read back with `social.content.artifact_get` and the `artifact_id` from create. `artifact.content` must equal the draft, `artifact.status` must be `draft`, and nothing is scheduled. A 404 means the id is not in this workspace: list again, and do not create a duplicate. An edit is `social.content.artifact_update` with `expected_revision` set to `artifact.revision` from that read. The patch clears approval (`approval_invalidated` true) and can return 409 `published_immutable` or 409 `stale_action`. Stop, re-read, and do not open a second artifact.

Beside the copy, report the hook family, the character count of line 1 and of the first 210 characters, the skim skeleton, withheld claims, and the readback.

## The fold is the whole bet

The feed shows about the first 210 characters (roughly 3 lines) before "see more". That window is the hook. It has two jobs:

1. Stop the scroll: the reader's own language (their role, goal or pain), a sharp specific (a number, a timeframe, an odd fact), a pattern break, or a scene they want to finish.
2. Earn the click: a bold claim or stat needs a bridge clause inside the window that promises a payoff. A story hook uses the unfinished scene as the promise. A question hook uses the open loop.

A hook that ends on a flat statement with no bridge, loop or tension is unfinished. Never invent a number to make a hook land; use only numbers the person gave you or the workspace knows.

## Structure and formatting

- Paragraphs of 1 to 4 sentences. Mix 3 to 5 word punches with 15 to 25 word explanations.
- Line breaks carry rhythm. A run of short lines with no blank line between them builds tension before a payoff line.
- Lists use `1.` or `•`, never `-` or `*`. Bold every list item title or none.
- No decorative separators (`---`, `___`, `***`). No em dashes, en dashes or double hyphens.
- One core idea per post. Active voice, contractions, "And", "But", "So" are fine. Never start 3 or more sentences in a row with "I".
- One specific, slightly imperfect detail per post reads as a real person.

## Length

The limit is 3,000 characters. Match length to the type: a short post can be under 100 characters; story, case study and list posts often run 600 to 1,500. Never pad to reach a length.

## Pick the post type by what the person has

For a vague LinkedIn request, use social's choice-based intake: offer story, how-to, opinion, announcement, or carousel. Do not ask when user already gave a specific type or enough concrete direction to choose one.

- **Story** (a real event with a turn): open on the outcome or the conflict, not setup. 2 to 3 short paragraphs of timeline with real markers, the conflict by paragraph 3, the specific resolution, then one line of reflection (never "The lesson is...").
- **List** (3 or more distinct items): open with the specific promise. Each item: number and name, what it does in one line, why it matters in 1 to 2 lines. Blank line between items.
- **Case study** (a before and after with numbers): the result in line 1 or 2, the before state, the one turning point decision, 2 to 4 replicable steps including one counter-intuitive one, the outcome with timeframe, one transferable principle.
- **Belief disruption** (the reader has the wrong diagnosis): state the common belief fairly in the reader's words, one concrete counter-example, the new frame in one sentence, what changes if they accept it. The disruption can be the close.
- **Short** (one sharp claim): 1 to 2 line hook that is the point, optional 1 to 3 line contrast, optional close. Each line its own paragraph.
- **Paragraph** (an argument to build, the default when unsure): 2 to 3 line tension opening, 2 to 5 short paragraphs of one idea each, the most quotable insight second to last, a specific close. For promoting something the person built, use problem, agitate, solution, and do not rush the agitation.

## Specificity is credibility

"$85k" beats "high pay". "10 days" beats "quickly". A named company beats "top companies". Every number, name and outcome traces to something the person said or the workspace knows. Realistic numbers read as "I could do that"; huge round ones read as "not for me".

## The close

Every LinkedIn post ends with a real call to action unless the person explicitly says no CTA. The server accepts a last sentence that opens on an ask verb, a question that says "you" or "your", or a stock question such as "Thoughts?" or "Agree?". Those stock questions pass `linkedin_cta_required` and still fail this skill. They fit any post and offer nothing back. The last line asks for a concrete reply and offers something in return: a quick take, a template, or an honest critique. "Drop your role below and I'll share what I'd change first" is strong. "What would you change?" is the line the server appends under `cta_policy` `"append"`. Do not use it as the close.

## Hashtags, emojis, mentions, links

No hashtags or emojis unless asked (one trailing ↓ is fine when the hook promises a list that follows). Mention a person or company only when the post is about them or thanks them, and resolve every mention before scheduling. Links in the body are fine when they are the point of the post.

## Media

A post carries up to 9 images or GIFs, or 1 video (mp4 or mov), or 1 PDF document shown as a swipeable carousel, never mixed. An animated GIF posts as an animated GIF.
Attach media with the social skill's upload flow.

A carousel is 5 to 9 pages, because this channel allows at most 9 images. Fewer than 5 beats is one image, not a short carousel. Do not pad pages. Order the pages as promise, problem, mechanism, proof, then the same call to action as the post. One beat per page. Delete a page the sequence does not need. Do not invent a number to fill an empty column. Page lines are the author's own words, not a smoother paraphrase. A PDF is still one document. Do not mix it with images.

## Never write

"delve into", "tapestry", "unlock your potential", "navigate the complexities", "in today's fast-paced world", "it goes without saying", "game-changer", "at the end of the day", "let me share", "it's important to note", "some might say", and three-item lists padded to sound complete. Compress clauses into strong verbs: "it dies in a cold message" beats "it falls apart the second it becomes a cold message".

## Editing a post

Change exactly what was asked and keep everything else verbatim, formatting and line breaks included. A new opening does not license a new CTA. "Tighten it" means cut and compress, not reinvent. Revise the existing draft instead of creating a near duplicate.

## Refused save and failed readback

`linkedin_cta_required` (422, message starts "LinkedIn drafts must end with a call to action.") means fix the close and call `social.content.artifact_create` again only if nothing was saved. If a row exists, `social.content.artifact_update` it. Do not set `cta_policy` to `append`.

A 404 from `social.content.artifact_get` means the id is not in this workspace. Call `social.content.artifact_list` before any second create. 409 `published_immutable` or 409 `stale_action` means stop, re-read, and do not open another artifact. If `artifact.content` does not match the draft, report the stored text. Update once with `expected_revision` from that read. `approval_invalidated` true means approval was cleared.

## Check it works

Read the saved draft back before reporting it. Confirm it is under the channel limit, the first 210 characters are a complete hook, the skim skeleton carries the argument, the close is a specific call to action, every mention is resolved and nothing contains an em dash. If you saved, `artifact.content` from `social.content.artifact_get` matches the draft.

## Find genre-matched examples

For writing references, search Inspiration LinkedIn posts with an AI filter description, not keyword guesses or stored genre fields. Pass a short content description such as `A personal or founder story with a narrative arc.` or `A practical how-to post teaching a repeatable process or tactic.` Keep `limit` at 25 or fewer. Results include only model-matched posts and carry `ai_filter_reason`; use that reason to confirm fit before borrowing structure. `ai_filter` is query-time only. Never claim posts have a permanent genre label.

## Worked example

The person supplied one fact: an opener change booked 4 meetings between 12 and 22 March, and they will paste both lines if someone comments OPENER. Company Context has no other number. The last two posts used a result hook, so this one does not.

Bad draft:

```
It's not the copy. It's the system.

Distribution is the moat.

Thoughts?
```

Fixed draft:

```
The old opener named the product. The new one named the missed reply.

That change booked 4 meetings between 12 and 22 March.

Comment OPENER and I will paste both lines.
```

The bad draft fails the negation-pivot rule, ends on a slogan, and uses "Thoughts?", which the server accepts and this skill does not. The fixed draft keeps the supplied window, puts the product below the fold, and ends on an ask that offers the lines. The hook family is before and after, not result. If the 4 meetings had not been supplied, the verdict would be REPAIR, not this draft.

Save with `social.content.artifact_create`: `content` is the fixed draft, `channels` is `["linkedin"]`, `status` is `"draft"`, `cta_policy` is `"refuse"`. Then `social.content.artifact_get`. `artifact.content` must equal the fixed draft.

Paid and outward runs follow the paid-run consent rule.
