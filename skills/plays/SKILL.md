---
name: plays
description: "Recommend and run one named, proven go-to-market play when the user asks what to do next, picking by stage and what the workspace already has, and stating prerequisites, credit estimate and checkpoints before starting. Use it for \"what should I do\", \"how do I get leads\", \"help me start outbound or content\", or a weekly review. It does not replace the skills each step uses (knowledge, list-building, outreach, social, writing, visibility, workflows)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Plays

## Boundary

- Owns: picking and running one named proven play.
- Diagnosing the constraint and designing a new motion: [`gtm-consultant`].
- The `sequence` pack lists the tools and their exact inputs.

Where each stage lives: the `tables` pack holds lists and signal capture, the `sequence` pack sends, the `unibox` pack holds replies, and the `knowledge` pack holds Company Context and play verdicts.

A play is a proven order of ordinary steps aimed at one business outcome. It teaches order and judgment. Each step still runs through its own skill and tools.

`first-win` is only an alias. A fresh workspace, or "get me started", runs the setup skill, then comes back here for one named play. Do not run the first-win steps yourself.

## Which skill does the work

- "What should I do?", a named play, or a weekly review: this skill. Return one play, not a menu.
- "Why is pipeline flat?", "what should we change?", or "plan our GTM": the gtm-consultant skill. It designs the motion. This catalog is evidence, not the diagnosis.
- One stage: the skill that owns it. Lists are [`list-building`]. Warm signals are [`linkedin-warm-signals`]. Sequences are [`outreach`]. Posts are the social skills. Replies in existing threads are [`inbox-replies`]. Automations are [`workflows`].

## The invariant

"What should I do?" gets one named play with a reason. When the user names a play, run that play. Shape it from Company Context, connected accounts, worksheets, sequences, drafts, and analytics. Lists, warm leads, and scores live in a worksheet. There is no separate leads feed. Never run two plays on the same audience.

## Choose one play

Name the binding constraint in one line from the reads below, then stop at the first matching play. Do not offer a menu. The next play waits until the check-in.

- No Company Context, or the user said get me started in an empty workspace: [`setup`]. `first-win` is only the alias. Do not install a play.
- The user is asking about replies already waiting: [`inbox-replies`]. Not a play.
- A motion has been live long enough to judge, or the user asked for a monthly review: Day-30 review.
- The user named a play: that play, even when another signal is also true.
- Constraint is posts or copy, and the question is what to publish: What should I post? If the question is what already worked: Analyze my social media. Not both in one pilot.
- Warm rows are not landing, and the gap is qualified conversations: Inbound-led outbound when a LinkedIn account can capture or the play can stop at a draft. TAM sourcing when there is no audience worksheet at all.
- Warm rows are already landing and there is no morning list: Signal-based outbound.
- A target worksheet exists and the user wants competitor engagers: People engaging with a prospect's competitors.
- Several signal worksheets describe the same companies and no single warm source is the constraint: Inferred pain from weak signals.

The signals that decide are: Company Context present or missing, replies waiting, a live sequence or posting habit old enough to judge, warm rows landing or not, a morning list present or not, a target account worksheet, and whether the user named a play. `sequence.outreach.analytics_performance_query` and `social.content.performance_query` tell you the constraint. They do not pick a second play.

A claim needs a number this workspace returned, or a named customer. Otherwise it is unproven. Do not borrow an industry rate and call it this account's result.

## Practice

1. **Name the constraint, then read.** Before the play, say the binding constraint in one line: not enough qualified conversations, no warm source, copy that does not earn replies, or no offer in Company Context. Read past results first, following the performance skill: `sequence.outreach.analytics_performance_query` for outreach (rates are people who responded over people messaged; a group under 30 people or `insufficient_data` is a hint) and `social.content.performance_query` for posts. Then `workspace.blueprints.recommend` with `kind` `workbook`, `sequence`, or `workflow`. It ranks shipped blueprints from connected accounts, worksheets, and credit posture, and it spends nothing. For one candidate, `workspace.blueprints.describe`, then `workspace.blueprints.preflight`. Preflight writes nothing and returns every missing prerequisite, or a plan whose automation is disabled. Read Company Context. Reuse a worksheet or sequence that already does the job. Do not install a second copy.
2. **State the plan.** Steps in order, what each produces, the credit posture, and the approval gate. State the estimate, then proceed under the paid-run consent rule. Tell the user what the gate allows before you ask for it: the exact cap, what it will contact or spend, and that install alone does not send.
3. **Write the pilot spec** in [Pilot before scale](#pilot-before-scale) before the first send. Then run step by step, loading each step's skill with skills_get. Build what you can when an account is missing, and name what to connect.
4. **Install only after preflight.** `workspace.blueprints.preflight` takes `blueprint_id`, `request_key`, and `params`. A result with `ok` false, a non-empty `missing_integrations`, or a non-empty `missing_context_keys` is an unmet prerequisite. Do not call `workspace.blueprints.apply`. Name each missing item. Write the Company Context or the draft copy that does not need the account, and say what to connect. Do not install a second blueprint to get around the gap. `spend.credits_used` on a preflight is 0.

   `workspace.blueprints.apply` takes `blueprint_id` and `inputs` keyed by the keys describe returned. It spends no credits, calls no provider, and sends nothing. A workbook blueprint creates sources and columns with nothing running until inputs are set. A sequence blueprint creates a draft that is never started. A workflow blueprint creates a disabled workflow draft. Applying does not enroll, publish, or start. `workspace.blueprints.configure` sets inputs later and still spends nothing. Producer-managed signal rows arrive from the producer, not from a manual poll. Sample them with `tables.rows.query`.

   If apply fails, read the error and stay on the same `blueprint_id`. `blueprint_not_found:<id>` means that id is not in `workspace.blueprints.list` or `workspace.blueprints.recommend`. Describe again. Do not invent an id. `inputs.<key> is not declared for blueprint <id>` or `inputs.<key> is invalid` means fix that key from `workspace.blueprints.describe` and apply again. Do not hand-build a replacement. A readback that shows credits above 0, a started sequence, or an enabled workflow is not a successful install. Stop and report the returned `kind`, `sequence_id` or `workflow_id`, and `web_url`. If the workbook exists and `configure` still lists unset `input_keys`, call `workspace.blueprints.configure`. Do not apply a second time to fill inputs.
5. **Show before it leaves.** Final copy of every message or post before anything sends or publishes. Say so before any daily schedule.
6. **Close with the leading metric** and the check-in date from the pilot spec. Log the verdict (scale, change, or stop) through the knowledge skill. A number you did not read back is not a verdict.

A recommendation spends nothing. Every step inside the selected play still uses that step's own preview and the paid-run consent rule.

## Pilot before scale

Write this down before the first send. One variable only (audience, opener, or channel, not all three).

- Success bar: the number that means continue, the window, and which read proves it (`sequence.outreach.analytics_performance_query` or `social.content.performance_query`).
- Revise band: the range where you change the one variable and run the pilot again.
- Kill line: the number, or the missing prerequisite, that stops the play.
- Guardrails: sender health, do-not-contact, and people already in a sequence. A guardrail breach stops the play even if the success bar is ahead.
- Kill switch: pause the sequence or the source schedule. Say which object you will pause.
- Check-in date.

Then log what happened. Queued or simulated is not a confirmed send. Report launched, accepted, confirmed, and replied as separate counts when the tool returns them.

## Catalog

Each play below states prerequisites, credit posture, approval, recovery, a success bar, and a kill line. Steps still follow the skill named in the step.

**Inbound-led outbound** (inbound-led-outbound). People already paying attention become conversations.

- Prerequisites: Company Context has an ideal customer. A LinkedIn account is connected, or the play stops at the draft. One worksheet source among reactions and comments on the user's posts, followers, or profile viewers. [`linkedin-warm-signals`] owns capture and fit.
- Credit posture: own-account capture is metered, not a credit quote. Fit and contact data are paid and follow the paid-run consent rule.
- Approval: show final copy, audience count, sender, and daily cap before enroll and start.
- Recovery: empty capture uses the warm-signal status read. Do not add a second source of the same kind.
- Success bar: qualified people per sample, then positive replies per ten messaged, on the check-in date.
- Kill line: replies are mostly failed-fit rows. Tighten Required criteria. Do not loosen fit to fill the week.
- Steps: one source per worksheet. `column.ai_fit_score` against the ideal customer, high, medium, or low, with a one-line reason. Sample first. Sequence only the high-fit view. The first message may name a public post. It never says you saw them react, follow, or view.

**Signal-based outbound** (signal-based-outbound). One ranked list each morning.

- Prerequisites: at least one warm source already landing rows. Ideal customer written.
- Credit posture: `crm.leads.today_list` is free. Refreshing a paid source is paid.
- Approval: say so before a morning schedule. The schedule is a hosted source or workflow, not a local cron.
- Recovery: `icp_fit` null means no stored score. Do not drop the person for that alone, and do not treat them as qualified.
- Success bar: warm people contacted from the list, and the replies they produce.
- Kill line: three mornings with no row that passes fit. Stop the enroll and fix the source or the rubric.
- Steps: `crm.leads.today_list` first (`limit`, `since_hours` up to 168 for a week). Post engagers and new followers before profile viewers. Fit is separate from how recent the signal was. Top five to ten, with what they did and when, and a first line from a true public fact.

**People engaging with a prospect's competitors** (competitor-engager-prospecting).

- Prerequisites: a target worksheet. Competitor LinkedIn URLs reviewed before spend.
- Credit posture: state the estimate. Pilot five targets. Later work runs only where fit is 70 or higher.
- Approval: review returned competitor URLs before the engager pull. Paid runs follow the paid-run consent rule.
- Recovery: if a target returns no engagers, say so. Do not widen to the whole company.
- Success bar: qualified competitor engagers per target, then replies.
- Kill line: after five targets, qualified engagers are near zero. Stop and change the competitor set. Do not drop the score floor.
- Steps: fit on the target worksheet. An AI or web column returns at most five competitor LinkedIn URLs. Run worksheet, post engagers in profile mode, once per reviewed URL. The child worksheet excludes competitor employees and existing customers, then scores fit. Return the top ten child rows, the count, and a share link. The share link lasts 30 days and can be revoked. Sequence step one states the count only, with no link. Step two sends the share link. Before a reply, a teaser may name person, title, company, and the public post. Never include email or phone.

**Inferred pain from weak signals** (signal-synthesis).

- Prerequisites: one signals worksheet per source, each row carrying a company LinkedIn URL or domain. Ideal customer and offer written.
- Credit posture: Lookup rows is free. The AI column and any employee fan-out are paid. Sample before the full run.
- Approval: read the sample with the user before every row runs.
- Recovery: the model says "none found" when the signals do not connect. Do not invent a pain to keep the row.
- Success bar: accounts with a high-confidence pain, then replies per ten messaged.
- Kill line: the sample pains are not pains a buyer would recognize. Stop and change the sources. Do not enroll the rest.
- Steps: Lookup rows on the company URL, multiple rows, newest first, limit about five. Company-page posts count only when they name a system, a migration, an acquisition, or a platform change. The AI column returns pain in one sentence, confidence high, medium, or low, the signals and dates, and the title of the leader who owns the fix. Fan out employees only for accounts with a pain. The opener names the pain and the public evidence. It does not say "we saw your job post" as a private sighting. Hiring, funding, and a public post may be named.

**What should I post?** (founder-posting-system).

- Prerequisites: a connected account with past posts, or an explicit note that voice has no sample yet. Tone of Voice in Company Context.
- Credit posture: drafting is not a send. Scheduling spends nothing until publish.
- Approval: one post at a time. Nothing publishes unapproved.
- Recovery: if past posts cannot be read, say which account failed. Draft from Tone of Voice and mark the voice as thin.
- Success bar: posts published per week, then comments from people who could buy, read with `social.content.performance_query`.
- Kill line: two weeks of posts and no comment from a buyer. Change the post type. Do not add a second channel in the same pilot.
- Steps: learn voice from posts that drew comments. Draft a few posts of different kinds, each with one concrete point, who it is for, and a call to action. After publish, read the result and write more like the posts that got comments.

**Analyze my social media** (social-performance-review).

- Prerequisites: at least one connected social account.
- Credit posture: `social.content.performance_query` is a read. State freshness before any conclusion.
- Approval: none for the read. A next-week experiment that publishes still needs the post approval.
- Recovery: missing or stale data names the account and the reason. Do not fill the gap with a hand count.
- Success bar: the metric the workspace optimizes for, week over week, from that read.
- Kill line: fewer than three prior posts. Say `not_enough_history` and do not declare a winner.
- Steps: totals and trend against the previous period, best and worst posts against the user's own median, follower change. Patterns need the posts that show them. End with three experiments. File one testable difference through the knowledge skill.

**TAM sourcing** (tam-sourcing).

- Prerequisites: ideal customer written, or a list the user already shared.
- Credit posture: company search and the fit column are paid. Preview and a small sample before the list of about 25.
- Approval: the paid-run consent rule on the sample, then again before you grow it.
- Recovery: a near-empty preview means the filter is too narrow. Loosen one filter once and preview again. Do not stack a second source to hide an empty preview.
- Success bar: about 25 companies graded with a reason, and a sentence on which kinds fit.
- Kill line: under about 10 percent graded as fit after one loosen. Stop and change the source definition with [`list-building`].

**Day-30 review** (day-30-review). Use when a motion has been live long enough to judge, or the user asks for a monthly review.

- Prerequisites: at least one live sequence, worksheet, or posting habit. The pilot spec from when it started.
- Credit posture: reads only. `workspace.usage.spend_breakdown`, `sequence.outreach.analytics_performance_query`, `social.content.performance_query`. Name only categories and counts the tools return.
- Approval: none for the reads. Pausing a live sequence needs a yes, and you say how to undo it.
- Recovery: a refused or empty analytics read is a missing result. Say so. Do not invent a rate. Account fees that the spend read shows as still running stay running until a separate pause exists.
- Success bar: the original pilot bar, compared with the number you just read.
- Kill line: the original kill line, or a guardrail breach. Verdict is scale, change, or stop. Log it.

Other motions when the state calls for them: setup on a fresh workspace, a reply path when replies are waiting, a visibility sprint for search and answer engines. Still one play.

## Choosing one result, a split, or a child worksheet

- Use a function when each parent row needs one value, such as one score, verdict, URL, or count.
- Use fan out for a one-off split where returned items become ordinary rows and no parent-scoped result must return.
- Use Run worksheet when each parent must produce bounded child rows, run a child qualification chain, then return ranked rows, a count, or a share link. The same person may appear under different parents.

## Guardrails

- Never run two plays against the same audience.
- Anything sent externally is shown with final copy before the send step.
- A play is not mandatory architecture. Skip or reorder a step whose output already exists.
- Installing a blueprint starts nothing. A disabled workflow stays disabled until a later, separate enable. A draft sequence stays a draft until publish and start.
- Do not tell the user to finish a step in the product UI when a tool can do it.

## Worked example

The user asks what to do next. Outreach has been live for a month. They have a LinkedIn account and no morning list.

1. `sequence.outreach.analytics_performance_query` for replies over at least 90 days, one audience dimension. Suppose the readback marks every group `insufficient_data`. Say that in one line. The constraint is "no qualified conversations yet", not "copy is bad".
2. `workspace.blueprints.recommend` with `{ "kind": "sequence" }`. Pick the one row that matches warm LinkedIn, not a second cold list. `workspace.blueprints.describe` with `{ "blueprint_id": "<id from recommend>" }`. `workspace.blueprints.preflight` with `{ "blueprint_id": "<id>", "request_key": "warm-pilot-1", "params": {} }`. A failed preflight lists missing prerequisites and no plan. Stop there until they are met.
3. Write the pilot before apply. Variable: audience (own-post engagers only). Success bar: 3 positive replies in 14 days among people who passed fit. Revise band: 1 or 2 replies, change the opener only. Kill line: 0 replies, or replies from failed-fit rows. Guardrail: no second sequence to the same people. Kill switch: pause that sequence. Check-in: day 14.
4. `workspace.blueprints.apply` with `{ "blueprint_id": "<id>", "inputs": {} }`. Readback: a draft sequence, credits 0, nothing started. If the blueprint is a workflow, the workflow is disabled. Configure inputs with `workspace.blueprints.configure` only after describe lists the keys.
5. Hand capture and fit to [`linkedin-warm-signals`], then enroll and start only after the user has seen final copy. The first line may name a public post. It may not name a profile view.

## What to return

- The play, the constraint, and why this play and not a second one.
- The pilot spec: success bar, revise band, kill line, guardrail, kill switch, check-in.
- The step plan with credit posture and the approval gate.
- As it runs: what each step produced, from readback, with links.
- Missing accounts, and the natural next play after the check-in. Do not start that play now.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered. Publish or send only when the user asked. An empty, failed, or wrong result means an input or setting is wrong: find which from the error or the output, correct it, and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
