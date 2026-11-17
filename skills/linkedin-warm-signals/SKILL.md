---
name: linkedin-warm-signals
description: "Warm people come from worksheet sources such as post engagers, followers, profile viewers and website visits. Judge worksheet rows against Required criteria, then build a capped warm sequence while protecting sender account. Use when user wants leads from LinkedIn presence or network-first motion. Does not cover cold lists from search (list-building does), writing posts (social does) or replying in existing threads (inbox-replies does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# LinkedIn warm signals

## Boundary

- Owns: capturing warm signals into worksheets, judging those rows, the who-to-contact-today read, and caps on warm sends.
- Sequence mechanics, copy, and pacing: [`outreach`].
- Cold lists from search sources: [`list-building`].
- The `sequence` pack lists the tools and their exact inputs.

The skill id stays `linkedin-warm-signals`. The scope is every warm signal, not only LinkedIn engagement. Capture lands in a worksheet. Reading a stored score or today's list is a CRM read. Enrolling people is a sequence step. Nothing in this skill sends a message by itself.

## When to use

Use this skill when the user wants people who already showed up: post engagers, followers, profile viewers, the connected network, new connections, hiring or funding searches, a profile watcher, product users, or a ranked list of who to contact today.

Do not use it for a cold list from search ([`list-building`]), for writing or pacing the sequence ([`outreach`]), for a reply in a thread that already exists, or for writing the user's own posts.

## The invariant

One signal source per worksheet. Person profiles (`/in/`) and company pages (`/company/`) are different sources. A strong signal never rescues a bad fit. Name a public signal in copy. Never name a private one.

## Safety ladder

1. Free read or preview first. `crm.leads.today_list`, `crm.records.score_explain`, `crm.signals.search_filters`, `crm.signals.search_plan`, `tables.findable_catalog.list`, `tables.profile_watcher.get`, `tables.profile_watcher.preview`, and the status reads below write nothing and charge nothing.
2. Dry run next. `crm.signals.search_run` defaults to dry run and writes nothing. `visibility.linkedin.intent_setup`, `visibility.linkedin.intent_autoenroll`, `visibility.linkedin.engagement_harvest`, and `visibility.linkedin.viewers_import` preview before a live call.
3. Small pilot. Calibrate fit on four rows the user can judge (a clear fit, a wrong company, a wrong person, missing evidence) before a paid batch. Run the paid batch under the paid-run consent rule.
4. Approval. A standing grant needs an explicit yes and a ceiling. State the recurring spend and who could be contacted. Raising the ceiling or changing the cadence asks again.
5. Ceilings. Keep `autoenroll_row_ceiling`, `watcher_max_credits`, `max_credits_per_cycle`, and the sequence's daily cap. A capped cell or a delayed send is pacing, not a failure.

Monitoring is hosted: a source schedule, a profile watcher, a bound feed, or a workflow. A local cron is not a monitor. Coverage is partial. Anonymous viewers stay hidden. Quote a count only when the tool returned it. Never say everyone.

## Read before you capture

Search what past outreach produced for this audience and channel, following the performance skill, and read that evidence before choosing the signal. Then read Ideal Customer and Tone of Voice.

Who to contact today is one free read, not a hand ranking:

```json
{ "limit": 10, "since_hours": 168 }
```

Call `crm.leads.today_list`. `limit` defaults to 50 and maxes at 200. `since_hours` defaults to 72 and maxes at 720. Use 168 for the last 7 days. People in an active sequence or on a suppression list are omitted. The call does not enroll, send, or score with a model.

Readback is `{ "rows": [ { "person": { "name", "linkedin_url", "email", "company" }, "signal": { "kind", "occurred_at", "source_ref" }, "reason", "icp_fit", "rank" } ] }`. `kind` is `reply`, `comment`, `repeat_engagement`, `profile_view`, `follow`, or `other`. `icp_fit` is the stored score, or null when none is stored. Null is not a fail and not a zero. `reason` is built in code. Do not invent a reason.

For a person who already has a record, call `crm.records.score_explain` with `{ "record_id": "<uuid>" }`. Read `score.grade` (A to F, or null), `score.signal_level` (`Burning`, `Hot`, `Warm`, `Cold`), `score.verdict`, `score.excluded`, and `stale`. Grade is fit. Signal level is recency. Never combine them into one number. A hot signal does not override a failed grade or an `excluded` label. When `stale` is true, or `score_state` is `queued` or `dead`, say the stored score lags and read `score_last_error`. Writing the rubric is a setup step (`crm.score_rubric.get`, then `crm.score_rubric.set` with `apply` false, then `apply` true), never a routine step of this skill.

## Sources

Confirm every `definition_id` with `tables.findable_catalog.list` before attaching it. If the catalog does not return the id, do not attach it. Attach with the tables source tools. One source per worksheet.

| Need | Definition or capability | Cost |
| --- | --- | --- |
| Engagers on the user's own posts | `source.engaged_with_account`, then `visibility.linkedin.intent_setup` | Metered on the connected account. Sends nothing. |
| Engagers on a company or competitor page | `source.engaged_with_company` | Third-party research. Credits. |
| Followers of a connected account | `source.linkedin_followers` or `visibility.linkedin.followers_import` | Metered on the account. `source.linkedin_followers` needs `target_url`. |
| Followers of someone else's company page | `tables.followers.order`, then `tables.followers.status` | Paid order. Not `visibility.linkedin.followers_import`. |
| Profile viewers | `source.linkedin_profile_viewers` or `visibility.linkedin.viewers_import` | Metered on the account. Anonymous viewers stay hidden. |
| Whole network | `source.linkedin_connections` via `visibility.linkedin.network_capture_setup` | Metered. Setup does not call LinkedIn. |
| New connections | `source.linkedin_new_connection` | Metered. |
| Hiring | `crm.signals.search_plan` with `family` `hiring`, then `crm.signals.search_run` on `source.linkedin_jobs` | Plan is free. Live run is paid. |
| Funding on a watch list | `crm.signals.search_plan` with `family` `funding` and `scope` `watch_list` | Market-wide funding has no route. Do not invent one. |
| Public posts about a topic | `crm.signals.search_preview` (free, `linkedin_posts` only), then `crm.signals.search_run` on `source.linkedin_posts_search` | Preview charges nothing. A live preview mode is refused. |
| Named profiles, posts and job changes | `tables.profile_watcher.preview`, then `tables.profile_watcher.create` | Create stores the watcher and does not contact LinkedIn. Later cycles spend under `watcher_max_credits`. |
| Product users | `source.posthog_cohort` or `source.stripe_customers` | See [`list-building`]. Name the product action, never the tracking. |

Own-account reads go through the connected account and are metered like sends. Use a bounded window. Third-party pages, hiring, funding, and follower orders spend credits. Do not route third-party research through the account to save credits.

`crm.signals.search_plan` takes `prompt`, `family` (`hiring`, `technology`, `funding`, `acquisitions`, `news`, `linkedin_posts`), `scope` (`market` or `watch_list`), and optional `keywords`, `countries`, `last_days`, `estimate`. It returns routes, dropped filters, a credit estimate, and a table blueprint. No provider call. `estimated_match_count` stays null. `no_default_provider_chain` means that family and scope have no route.

`crm.signals.search_run` params:

```json
{
  "definition_id": "source.linkedin_jobs",
  "input_bindings": {},
  "table_id": "33333333-3333-4333-8333-333333333333",
  "approved": false,
  "max_pages": 1
}
```

`input_bindings` may contain only inputs that definition declares. Dry run is the default. Readback: `mode` `dry_run`, `credits_charged` 0, `definition_id`, `admission` `ready` or `blocked`, `blocked_by`, `source_id`, `run_id`, `feed`, `web_url`. A live run needs `approved` true, a credit ceiling, and a claimed step, under the paid-run consent rule. `bind_feed` true also needs `every` (`hourly`, `every_6h`, `daily@H`, or `weekly@mon9`, hours 0 to 23 UTC) and `max_credits_per_cycle`.

## Fit, then rank

Company fit and person fit are separate Required criteria. Both must pass. Engagement, its date, and the signal type are not criteria and never raise a failing row to a pass. Hard exclusions (competitor, existing customer, open conversation, the author's own team) override any strong signal. Details, the four-row calibration, and the `column.ai_fit_score` rubric are in [qualification.md](qualification.md).

Exclude with a Required criterion and run conditions, not by hand. Decide "existing customer" from Stripe on the email or company domain, as [`list-building`] describes, not by name.

## Act

Audience caps, the one nudge, and kill lines are in [audiences.md](audiences.md). A warm sequence is three steps or fewer. Build it with [`outreach`].

Copy names a public signal: hiring, funding, a public post, or a job change. Never name a profile view, a website visit, or a private engagement such as a reaction or a follow the person did not post. Quote a captured public comment or leave it out. Never invent what someone said.

Enrollment is `sequence.sequences.enroll_selection` with `sequence_id`, `worksheet_id`, and the qualified `view_id`. It enrolls the view's rows, spends no credits, and sends nobody. `created` 0 on an empty worksheet is not an error. A published LinkedIn sequence with no sender returns `sequence_no_senders` and stages zero rows. Start is `sequence.sequences.start`, after `sequence.sequences.publish` if the sequence is still a draft. `sequence_not_published` means publish or `sequence.sequences.resume` first.

Standing grants, each with an explicit yes and a ceiling:

- Row landing. `visibility.linkedin.intent_setup` first (`worksheet_id`, `engagement_kinds` of `reaction`, `comment`, or both). Then `visibility.linkedin.intent_autoenroll` with `enabled` true and `autoenroll_row_ceiling` (0 to 100000; 0 stops further rows). Dry run writes nothing. This ceilings engager rows. It does not message anyone.
- Future enrollment. `sequence.sequences.enroll_selection` with `live_follow` true re-binds that sequence to the qualified view, so later rows that enter the view stage too. Say who could be contacted and the sequence's daily cap before the yes. It still sends nobody until start.
- Profile watcher. Preview, then create with `approved` true, the unchanged `preview_hash`, and `watcher_max_credits`. `tables.profile_watcher.get` returns `watcher` null when the worksheet has none. Pause with `tables.profile_watcher.pause`.
- Bound search. `crm.signals.search_run` with `bind_feed` true only after the user accepts `max_credits_per_cycle`.
- Viewer enrichment. `visibility.linkedin.viewers_enrichment` only after a dry run shows the per-person price and the user accepts it. It does not contact people.
- Follower order. `tables.followers.order` with `company`, `credential_mode`, `request_id`, and `quoted_order_cost`. Reuse `request_id` after a timeout. Read the ledger with `tables.followers.status`. Budget exhaustion lands no partial rows.

One person, one sender. Never touch the same person from two accounts or two sequences. A reply, decline, or unsubscribe stops that person on every channel.

## When something fails

Read [capture-recovery.md](capture-recovery.md). Do not start a second source on top of a paused or stalled one. Fix the returned code and read the status again.

## Worked example

The user asks who to contact from last week's warm signals, then wants hiring companies if the list is thin.

1. `crm.leads.today_list` with `{ "limit": 10, "since_hours": 168 }`. Suppose the readback is one row: `rank` 1, `signal.kind` `comment`, `icp_fit` null, `reason` the code-built sentence. Null fit means no stored score. Do not treat the comment as qualification.
2. The row has a person record. `crm.records.score_explain` with `{ "record_id": "11111111-1111-4111-8111-111111111111" }` returns `score.grade` `F`, `score.signal_level` `Hot`, `score.excluded` null, `stale` false. Hot does not rescue the F. Do not enroll this person.
3. The list has nobody who passes. `crm.signals.search_plan` with `{ "prompt": "US companies hiring a head of sales", "family": "hiring", "scope": "market", "keywords": ["head of sales"], "countries": ["US"], "last_days": 30, "estimate": true }`. Read `routes`, `dropped` filters, and the credit estimate. If the plan says there is no route, stop.
4. Dry run `crm.signals.search_run` with the params in Sources, `definition_id` set to the id the plan returned (expect `source.linkedin_jobs` when the catalog still lists it). Readback `admission` `ready`, `credits_charged` 0, `mode` `dry_run` means nothing was written. A live run waits for the paid-run consent rule.
5. After rows land, add the two-sided fit in [qualification.md](qualification.md), read the four calibration rows, then enroll only the qualified view. The first line may name the public job post. It may not say the person viewed a profile or visited the site.

## What to return

- Signal, window, worksheet link, rows captured, qualified count, and one example reason from the cells.
- Who was left out and why (failed fit, exclusion, active sequence, suppression).
- Sequence steps with final copy, sender, daily cap, and whether any standing grant is armed.
- State proven from readback: built, enrolled, started. Credits charged. Accounts still to connect.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered. Publish or send only when the user asked. An empty, failed, or wrong result means an input or setting is wrong: find which from the error or the output, correct it, and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
