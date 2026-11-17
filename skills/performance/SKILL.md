---
name: performance
description: "Search what past outreach produced before choosing an audience or writing: pick the objective, one or two dimensions and a window of at least 90 days, read rates as people who responded over people messaged, treat small or immature groups as hints, then state the evidence and decide. Use it before choosing an audience, drafting a sequence step, changing a running sequence, or at a weekly review. It is not a rule to follow and never a message to copy (outreach builds sequences, list-building builds the audience)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Outreach performance

## Boundary

Owns: reading what past outreach produced before choosing an audience or writing.
- Sequence building and enrollment: [`outreach`].
- Building and qualifying who to contact: [`list-building`].
The `sequence` pack lists the tools and their exact inputs.

Every workspace that has sent outreach has a record of who was messaged, how, and who responded. Search it before you pick an audience, draft a step or change a running sequence, so you never start blind. Nothing here is forced: search, read the evidence, then decide.

## When to search

- Before choosing an audience for a new list or sequence.
- Before drafting a sequence step, a first message or a follow-up.
- Before changing a sequence that is already running.
- At a weekly review, to see what moved.

Skip it when the user has already told you exactly who to reach and what to say.

## How to ask

Use `sequence.outreach.analytics_performance_query`. Read its schema first for the exact inputs.

- Objective: reply, positive_reply, connection_accepted, meeting or deal. Pick the one that matches the goal of the work. A first message is usually judged on reply or positive_reply, a connection request on connection_accepted.
- Delivery health: the bounce and unsubscribe objectives are rates over people sent an email, so also filter surface to email. Use them to check whether a sequence is hurting the sender, not to judge copy.
- Dimensions: one or two only, so each group keeps enough people. Audience: title_seniority, title_function, company_industry, company_size_band, company_country, region, lead_source_family. Message and timing: channel, surface, length_bucket, has_question, step_number, touch_ordinal, channels_so_far, framework_id (the opener framework an AI-written step used; recorded from the day it shipped, so earlier sends group under null and read as not recorded). Sender: sender_account_id, also usable as a filter. Sequence: sequence_id, to compare sequences (null means the touch was not sent by a sequence); also usable as a filter. Sequencing: previous_action (what the person was sent or shown just before), days_since_previous (days since that earlier action), for questions such as whether liking a post before a connection request helps.
- Trend over time: sent_day, sent_week (ISO week, named by its Monday) or sent_month, all UTC, grouped from the touch's sent time. Use one of them to see whether results are improving, with a window long enough that each bucket holds 30 people.
- Compare steps: step_id, to see which step of a sequence earns the replies. Add a sequence_id filter so the steps belong to one sequence.
- People engaged: set include_engagement to true to count every person who received any touch, likes and profile views included, and read people_messaged. Never use it for a reply rate. The result also carries messages_sent, the number of touches in the cohort.
- Reply rate: set message_surfaces_only to true so the rate is people who replied over people who were sent a message they can reply to. Connection requests and likes are left out and are read with the connection_accepted objective instead.
- One person or one company's people: to see the history with one person, filter by contact_id or record_id and group by surface or step_number. Counts (messages_sent with include_engagement, people_with_outcome) are returned for one person; a rate on fewer than 30 people is still insufficient_data. These are filters only, never group_by, and they work in workspace scope without include_cost. A contact_id also matches the rows of that contact's canonical record, and a record_id matches its contacts.
- Time window: at least 90 days. A shorter window rarely has enough finished conversations to read.

To compare two versions of the same step, use `sequence.outreach.analytics_variant_compare`.

## How to read it

- A rate is the people who responded divided by the people messaged. It counts people, not messages.
- A group marked insufficient_data, or with fewer than 30 people, is a hint, not a finding. Say so and do not build on it.
- Recent cohorts read low because people have not had time to answer yet. Do not compare a young cohort with an old one.
- Versions that were not split at random cannot be called winners. If the same audience was not randomly divided between them, report the difference as an observation only.
- A big gap across several groups of real size is worth acting on. A small gap on a few people is noise.

## What to do with it

- State the evidence and the sample size to the user in one line, for example: "Replies were 9% for heads of finance (212 people) against 3% for founders (180 people) over the last 120 days." Then choose.
- When there is no data, or the groups are all too small, say so in one line and continue from Company Context and the ideal customer profile.
- When a finding changes what you would build, use it in the build and say which finding shaped which choice. Reading is free. Acting on a finding with a paid run follows the paid-run consent rule, so quote the cost first.

## What it is not

- It is not a rule. Evidence informs a decision, it never replaces the user's goal or Company Context.
- Never copy another message because it did well. Learn what kind of audience, channel and length responded, then write fresh copy for this request.
- Results describe this workspace and its own past sends, not every market or every sender.

## Network evidence

Set `scope` to `network` to see how other workspaces do on the same kind of outreach. Treat that as a prior, weaker than this workspace's own results. It comes back as bands, not exact rates, and it may say "not enough data yet", which is a normal answer: say so in one line and continue from this workspace's own results. When the network row includes this workspace's own counts, compare the two and lead with the workspace's own number. Network evidence never names another workspace and is never a reason to copy another sender's message.

## Posts

The same evidence habit applies to content. Before drafting a post, check what has worked for this account on this channel (format, length, hook, posting hour) with `social.content.performance_query`, and treat a small sample as a hint.

## Worked example

The user asks who to target next, and wants the reply evidence before any audience is chosen.

`sequence.outreach.analytics_performance_query` with:

```json
{
  "scope": "workspace",
  "from": "2026-08-18T00:00:00Z",
  "to": "2026-11-16T00:00:00Z",
  "group_by": ["title_function"],
  "objective": "positive_reply",
  "message_surfaces_only": true,
  "top_n": 25
}
```

`from` and `to` are ISO timestamps with an offset. The window is 90 days. One dimension. `message_surfaces_only` true keeps the rate as people who replied over people who were sent a message they can reply to.

Readback: each row has the people messaged and the rate. Say one line, for example "Positive replies were 9% for finance (212 people) against 3% for founders (180 people) from 2026-08-18 to 2026-11-16." A row marked `insufficient_data`, or under 30 people, is a hint. Do not recommend an audience from it. Then choose, and say which finding shaped the choice. Do not copy a past message. Omit `include_cost` unless the question is cost. When it is true, the row also carries `credits_spent`, `cost_per_outcome`, and `cost_coverage`.

A post question uses `social.content.performance_query`:

```json
{
  "from": "2026-08-18T00:00:00Z",
  "to": "2026-11-16T00:00:00Z",
  "group_by": ["hook_type"],
  "metric": "engagement_rate",
  "at_age": "7d",
  "top_n": 25
}
```

Readback: `rows[].posts`, `rows[].value`, `rows[].per_post`, `rows[].insufficient_data`, plus `posts_in_window`, `posts_without_metrics`, and `not_recorded`. A hook with no stored label is null. A metric named in `not_recorded` is absent. Do not fill it. A small `posts` count is a hint. `at_age` `7d` compares posts at the same age.

## If it goes wrong

| What came back | What to do |
| --- | --- |
| The query is refused | `tools_get` the schema. Fix the named field. `from` and `to` must be ISO timestamps with an offset. `group_by` is one or two dimensions. `objective` is one enum value. Do not invent a rate. |
| Rows are empty, or people messaged is 0, or `posts_in_window` is 0 | Say there is no evidence in that window. Widen the window once. Outreach stays at least 90 days. Drop one filter if you added one. Missing evidence is not a zero rate. |
| Every group is `insufficient_data` | Say so in one line. Continue from Company Context and the ideal customer. Do not pick a winner. |
| `scope` `network` comes back with not enough data yet | Say so. When the row includes this workspace as `own_workspace`, lead with that number. |
| Two variants were not split at random | `sequence.outreach.analytics_variant_compare` may still return rates. Report the people behind each rate as an observation. Do not call a winner. |
| `not_recorded` lists the metric you asked for | That metric is absent. Choose a metric the schema lists. Do not hand-count. |
| Post metrics look stale | This call has no `fresh` flag. State the window and `at_age`. Do not fill the gap with a hand count. |

## Cost

When the question is about cost, the outreach results query `sequence.outreach.analytics_performance_query` can return credits spent and cost per outcome with `include_cost`.
