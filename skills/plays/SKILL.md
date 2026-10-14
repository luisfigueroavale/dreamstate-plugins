---
name: plays
description: "Recommend and run one named, proven go-to-market play when the user asks what to do next, picking by stage and what the workspace already has, and stating prerequisites, credit estimate and checkpoints before starting. Use it for \"what should I do\", \"how do I get leads\", \"help me start outbound or content\", or a weekly review. It does not replace the skills each step uses (knowledge, list-building, outreach, social, writing, visibility, workflows)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

# Plays

## Boundary

- Owns: picking and running a named proven play.
- First session in a fresh workspace: [`first-win`].
- The `sequence` pack lists the tools and their exact inputs.

A play is a proven order of ordinary steps aimed at one business outcome. It teaches order and judgment; each step still runs through its own skill and tools with their normal rules.

## Backing packs

A play has no pack of its own. It routes across packs: `knowledge` for Company Context, `tables` for lists, `sequence` for outreach, `unibox` for replies, `social` for posts, `visibility` for analytics, `workflow` for automation. Use packs_list to see which are available, then packs_get only for the pack the current step needs.

## The invariant

"What should I do?" gets one named play with a reason, not a menu. When the user names a play, run that play. The play is shaped by the workspace's real state (Company Context, connected accounts, existing worksheets, sequences, drafts and analytics), not a generic default. Lists, warm leads and scores always live in a worksheet: a row source, evidence columns, an AI judgment column and a score. There is no separate leads feed.

## Practice

1. **Read the workspace.** Search first for three free, state-aware blueprint recommendations. That read uses registry requirements plus connected LinkedIn accounts, mailboxes, worksheets and credit posture. For one candidate, read the exact describe result the recommendation names next. Then read Company Context (Ideal Customer, Offer, Tone of Voice), connected LinkedIn and X accounts, existing worksheets, sequences, drafts and analytics freshness. Reuse what exists; never rebuild a worksheet or sequence that already does the job.
2. **State the plan.** Steps in order, what each produces, the credit estimate for paid steps, and the checkpoints where the user sees real output. State the estimate, then proceed under the paid-run consent rule.
3. **Run it step by step,** loading each step's skill with skills_get. Build everything buildable even when an account is missing, then name what to connect.
4. **Show before it leaves.** Final copy of every message or post before anything sends or publishes, and say so before switching on any daily schedule.
5. **Close with the leading metric** and when to check it.

## Catalog

**Inbound-led outbound** (inbound-led-outbound). People already paying attention become conversations.
- Worksheet sources: reactions and comments on the user's own recent posts or on posts by people they pick, followers and connections, profile viewers. One source per worksheet.
- AI judgment column: fit against the Ideal Customer page, level high, medium or low with a one-line reason from the headline and company. Run a sample first, read the top rows with the user.
- Sequence to the high-fit rows only, chosen by a column filter, a few new people a day, already-contacted people excluded. The first message opens on the shared topic and never says you saw them react, follow or view.
- Metric: good fits per sample, positive replies per ten messaged.

**Signal-based outbound** (signal-based-outbound). One ranked list each morning of the warmest people engaging with the user's content.
- One worksheet with the engagement sources: post engagers and new followers first, then profile viewers when connected.
- AI judgment column scoring fit plus how recent and how strong the engagement was; sort best first.
- Refresh every morning so new people land and only new rows run the criterion columns. Say before switching it on.
- Each morning: the top five to ten with what they did, when, and an honest first line built on a true fact.
- Metric: warm people contacted from the list, replies they produce.

**What should I post?** (founder-posting-system). Regular LinkedIn posts in the user's own voice.
- Learn the voice from the account's past posts; posts that drew comments are the best examples.
- Draft a few posts or plan a week of different kinds (customer story, contrarian take, build in public), each with one concrete point, who it is for, and a call to action.
- Schedule each for the user to approve one by one; nothing publishes unapproved.
- After publishing, read back how each did and write more like the ones that got comments.
- Metric: posts published per week, then comments from people who could buy.

**Analyze my social media** (social-performance-review). What works and what does not.
- Read stored analytics first and state freshness; if data is missing or stale, say which account and why.
- Totals and trend over the period against the previous one, best and worst posts against the user's own median, follower change.
- Patterns: themes, formats, length, hooks, posting days and times that win or lose, each backed by the posts that show it.
- End with three concrete experiments for next week.
- Metric: the metric the workspace optimizes for, week over week.

**TAM sourcing** (tam-sourcing). About 25 companies that look like the best customers, graded with a reason, or a list the user shares graded instead. Company search or the shared list as the source, an AI fit grade, then name which kinds of company fit best.

Other plays when the state calls for them: first win on a fresh workspace, reply to meeting when replies wait, visibility sprint for search and answer engines, weekly review of any live motion.

## Guardrails

- Never run two plays against the same audience; the same person would be touched twice.
- Anything sent externally (posts, DMs, emails) is shown with final copy before the send step.
- A play is not mandatory architecture. Skip or reorder steps whose output already exists.

## What to return

- The play and why, the step plan with credit estimates.
- As it runs: what each step produced, with links, read back from results.
- Missing accounts, the leading metric to watch, and the natural next play.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
