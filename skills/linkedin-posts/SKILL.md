---
name: linkedin-posts
description: "Write and revise LinkedIn posts that earn the click and the reply: the 210 character hook, structure and formatting, length by post type (story, list, case study, belief disruption, short, paragraph), specificity, the CTA close, and phrases to avoid. Use it for the copy of any post going to LinkedIn. It does not cover scheduling, publishing or automations (social does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# LinkedIn posts

## Boundary

- Owns: LinkedIn post copy.
- Post lifecycle, review, and publishing: [`social`].
- The `social` pack lists the tools and their exact inputs.

Write every LinkedIn post so it survives two moments: the feed preview and the close. Everything between them keeps the promise the first three lines made. Voice comes from the account's past posts and brand voice; structure comes from the post type below.

Before drafting, inspect collected inspiration examples when a user names a genre, topic, author, or reference style. Use only examples available to the active workspace, treat their text as structure evidence rather than copy, and say when the library has no matching classified genre. Browse curated templates when the user wants a proven structure, then adapt the selected template to the workspace evidence and voice. A template is guidance, never proof for a factual claim and never permission to copy an example.

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

Every LinkedIn post ends with a real call to action unless the person explicitly says no CTA. The last line opens with an ask verb or is a question addressed to the reader, and it offers something back: a quick take, a template, an honest critique. "Thoughts?" and "Agree?" fit any post, which is why they are weak. "Drop your role below and I'll share what I'd change first" is strong.

## Hashtags, emojis, mentions, links

No hashtags or emojis unless asked (one trailing ↓ is fine when the hook promises a list that follows). Mention a person or company only when the post is about them or thanks them, and resolve every mention before scheduling. Links in the body are fine when they are the point of the post.

## Never write

"delve into", "tapestry", "unlock your potential", "navigate the complexities", "in today's fast-paced world", "it goes without saying", "game-changer", "at the end of the day", "let me share", "it's important to note", "some might say", and three-item lists padded to sound complete. Compress clauses into strong verbs: "it dies in a cold message" beats "it falls apart the second it becomes a cold message".

## Editing a post

Change exactly what was asked and keep everything else verbatim, formatting and line breaks included. A new opening does not license a new CTA. "Tighten it" means cut and compress, not reinvent. Revise the existing draft instead of creating a near duplicate.

## Check it works

Read the saved draft back before reporting it. Confirm it is under the channel limit, the first 210 characters are a complete hook, the close follows the CTA rule, every mention is resolved and nothing contains an em dash.

## Find genre-matched examples

For writing references, search Inspiration LinkedIn posts with an AI filter description, not keyword guesses or stored genre fields. Pass a short content description such as `A personal or founder story with a narrative arc.` or `A practical how-to post teaching a repeatable process or tactic.` Keep `limit` at 25 or fewer. Results include only model-matched posts and carry `ai_filter_reason`; use that reason to confirm fit before borrowing structure. `ai_filter` is query-time only. Never claim posts have a permanent genre label.

Paid and outward runs follow the paid-run consent rule.
