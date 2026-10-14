---
name: linkedin-warm-signals
description: "Warm people come from worksheet sources such as post engagers, followers, profile viewers and website visits. Judge worksheet rows against Required criteria, then build a capped warm sequence while protecting sender account. Use when user wants leads from LinkedIn presence or network-first motion. Does not cover cold lists from search (list-building does), writing posts (social does) or replying in existing threads (inbox-replies does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. A larger paid run also needs the user's yes before you send approved: true; never send it on your own. If the server asks you to sign in, approve it in the browser.

# LinkedIn Warm Signals

## Boundary

- Owns: judging warm-source rows and warm send caps.
- Sequence mechanics, copy, and pacing: [`outreach`].
- Cold lists from search sources: [`list-building`].
- The `sequence` pack lists the tools and their exact inputs.

People who already engaged are the warmest audience a business has. Capture the signal into a worksheet, keep only the ones who fit, and reach them with a message that names the signal honestly, at a pace the account can sustain.

## Backing packs

The tables pack backs the signal worksheet: its source and criterion columns. The sequence pack backs the warm sequence and LinkedIn engagement actions. The integrations pack holds the LinkedIn account connection and signal source configuration. Public comments on posts are observations in the visibility pack.

## The invariant

Signals land in a workbook worksheet as a row source, with criterion columns marked Required or Nice to have, like any list. Outreach goes through a sequence of LinkedIn message and email steps, paced by its caps and sending hours. Account safety is a rule, not a suggestion.

## Practice

1. **Read context.** Ideal Customer and Tone of Voice in Company Context. Confirm which LinkedIn account is connected.
2. **Pick one signal.**
   - Engagers on the account's own posts: highest intent, reference the post.
   - Profile viewers and new followers: warm but softer; lead with relevance.
   - Engagers on a competitor or peer post: public research, on its own worksheet.
   - Sort pasted LinkedIn URLs first: person profiles (/in/) and company pages (/company/) are different sources. Build one source per kind on its own worksheet and say which went where.
3. **Build the worksheet** (list-building skill). One signal source per worksheet. Columns: the signal (which post, what they said), an AI fit judgment against the ICP with the supporting quote as a Required criterion column, then contact data gated on the Required criteria.
4. **Run a sample and read the rows.** Exclude the author's team, existing customers and open conversations with a Required criterion column and run conditions, not by hand.
5. **Draft the warm sequence** (outreach skill). The first LinkedIn message names the real signal in one line, adds one relevant point, and asks a light question. Three steps or fewer; warm people do not need a long cadence.
6. **Show and start.** Show the final copy of each step, audience count, sender and expected pace in days, then enroll and start.
7. **Recurring, if asked.** A workflow adds new signal rows on a schedule and enrolls those that pass every Required criterion.

Search the tool for each task. The search returns runnable schemas for its top hits; read any other ids together in one tools_get, then run.

## Safe limits

- Connected accounts read what they own: their posts, network and inbox. Third-party profiles and competitor posts are researched with tools that do not act through the account, and those cost credits. Routing third-party research through the account to save credits is how accounts get restricted.
- Reads are metered like sends. Use a bounded window, not the whole network or inbox.
- One person, one sender. Never touch the same person from two accounts or two sequences.
- A capped cell or delayed send is pacing, not a failure.
- A reply, decline or unsubscribe stops that person on every channel.

## Guardrails

- Name public engagement only; never write as if you watched someone.
- Quote the captured comment or leave it out. Never invent what someone said.
- Show the estimate for paid research and contact data, then run it under the paid-run consent rule: a run above the server's row or provider-spend limit needs the user's yes first.

## What to return

- Signal, window, worksheet link, rows captured, qualified count with example reasons.
- Warm sequence steps with final copy, sender, cap and expected pace.
- State proven (built, enrolled, started), credits charged, accounts to connect.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
