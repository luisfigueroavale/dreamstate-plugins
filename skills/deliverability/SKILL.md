---
name: deliverability
description: "Plan and protect email sending: mailbox and domain sizing, sending domains and their readiness, warm-up, mailbox health, deliverability tests, bounce and suppression evidence, and when to rest or retire a mailbox. Results are directional."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Deliverability

## Boundary
- Owns: Mailbox health, sending domain readiness, placement evidence, suppression, and email infrastructure decisions.
- The `senders` pack lists the tools and their exact inputs.

Use this for sender health, warm-up, placement, domain DNS, mailbox capacity, import, purchase, cancellation, rotation, or suppression. Designing or launching a sequence belongs to [`outreach`]. A stuck sequence run belongs to [`diagnostics`]. Choosing the GTM constraint belongs to [`gtm-consultant`].

## Route

| Job | Start here | Do not start here |
| --- | --- | --- |
| Triage a silent or bouncing inbox | `senders.mailboxes.health_get`, then `senders.mailboxes.list` | A placement send |
| Is the domain safe to send? | `senders.domains.sending.list`, then `senders.domains.sending.readiness_get` | Buying another domain |
| What did we already measure? | `senders.deliverability.evidence_list` | A new test |
| Will this copy land between our own mailboxes? | `senders.deliverability.workspace_test_preview` | `senders.mailboxes.placement_test` |
| Seed folders for one mailbox | `senders.mailboxes.placement_test` after the preview ladder below | A test on a paused mailbox |
| Monitor inboxes | `senders.mailboxes.deliverability_connect` preview | Arming auto-enroll before the preview |
| New domain, DNS, inboxes, OAuth, import, wind-down | [Infrastructure](infrastructure.md) | Attaching capacity to a sequence first |

Placement and reputation evidence is advisory. It never pauses, unpauses, or changes a cap by itself. Confirm pause state and the reason on the mailbox before recommending rest, repair, or retirement. Bounce auto-pause is authoritative when health reports it. A reputation score is not an automatic pause, and this catalog does not publish a pause threshold to infer.

## Safety ladder

1. Free local reads: `senders.mailboxes.health_get`, `senders.mailboxes.list`, `senders.domains.sending.list`, `senders.domains.sending.readiness_get`, `senders.domains.sending.dns_plan`, `senders.mailboxes.rotation_get`, `senders.suppressions.list`, `senders.mailboxes.blocklist_get`, `senders.copy.spam_score`, `senders.deliverability.evidence_list`, `senders.deliverability.evidence_compare`.
2. `senders.mailboxes.warmup_status` reads the warm-up rail and stores `warmup_truth`. It does not send campaign mail. It can write a runaway breaker when dispatch attempts far exceed sends. Treat it as a state-changing read. Call it when warm-up truth is the question, not on every diagnosis.
3. Previews that do not send or charge: `senders.deliverability.workspace_test_preview`, `senders.mailboxes.purchase_preview`, `senders.mailboxes.add_inboxes_preview`, `senders.mailboxes.cancel_preview`, `senders.domains.check`, `senders.domains.purchase_preview`, and the preview mode of `senders.mailboxes.deliverability_connect`. `senders.mailboxes.import` with validate-only or dry run writes nothing.
4. External or paid effects follow the paid-run consent rule: workspace copy send, seed placement test, domain add and apply, mailbox purchase, add-inboxes, cancel, warm-up enable or disable, monitoring connect, and auto-enroll. Dry-run, read the quote, then live with that quote. Start with the smallest sender set that can change the decision.
5. Never lift `senders.suppressions.list` entries to force a send. Removal is a separate, explicit decision on the suppression capability that matches the entry, after you read why it exists.

Label every next step: read, free preview, settings write, external send, or paid enrollment. A zero-credit action can still send mail or change settings.

## Triage

`senders.mailboxes.health_get` returns compatibility, connection, trailing-24-hour volume, trailing-7-day bounce rate, one repair action per mailbox, and fleet totals. The bounce window is the same window that drives auto-pause. Read stored readiness, blockers, next actions, and warm-up truth on each mailbox.

Freshness: current means a trusted measurement at most 24 hours old, stale means older evidence, unmeasured means no trusted measurement time. A recent poll time is not a new measurement. Sync `ok` includes an empty successful read. `blocked` and `failed` are not clean. `unknown` proves neither success nor a blocker. Zero sequence sends means no measured campaign exposure. Zero rejections is not a placement clearance.

Then, only as the hypothesis needs them: `senders.domains.sending.readiness_get` for SPF, DKIM, DMARC, MX, ESP verification, domain age, warm-up, mailbox authentication, and daily capacity. `senders.suppressions.list` and `senders.mailboxes.blocklist_get` are different lists. The blocklist is one mailbox's projection rules. Suppressions are the workspace do-not-contact ledger. Neither changes what is sent.

Form one hypothesis. Pick the cheapest check that could change the decision. Read stored evidence before sending another probe.

## Placement

Read `senders.deliverability.evidence_list` first. Kinds include `workspace_copy`, `seed_placement`, `copy_score`, and `dns_check`. An empty list is unmeasured, not inbox. Compare two rows of the same kind with `senders.deliverability.evidence_compare`. An empty sample has a null rate, not zero.

Score copy for no send with `senders.copy.spam_score` (subject, body, and `bulk` when the schema asks). Use real prospect copy. Do not send a delivery-check or no-reply body.

Workspace copy, when the question is Gmail labels or Microsoft Focused versus Other on connected workspace mailboxes: `senders.deliverability.workspace_test_preview` with `sender_ids`, `receiver_ids`, `subject`, and `body_text`. It checks that those ids belong to this workspace and returns the copies and `proposed_sends`. It does not send and does not pause anyone. `credit_estimate` on that preview is zeros. The live `senders.deliverability.workspace_test_run` still leaves the workspace and follows the paid-run consent rule because it sends. Read `observations`, `by_provider` when present, and `paused_any_mailbox` (always false). Then list evidence again and compare.

Seed placement, when the question is inbox, spam, promotions, other, or missing on outside seeds: `senders.mailboxes.placement_test` with `mailbox_id` from `senders.mailboxes.list`, `subject`, and `body_text`. Optional `seeds` stay inside the schema max. The probe is refused when that mailbox is paused (`refused_paused_mailbox`). `unavailable` means no provider or a missing mailbox. `completed` is the only status with folder counts. The result is directional. It never unpauses a bounce-paused mailbox and never changes warm-up or ramp.

Monitoring price is on `senders.mailboxes.deliverability_connect`: the catalog states managed mode is 200 credits per inbox-month, with 2 placement tests per inbox per cycle included, and an extra managed placement test at 132 credits. Own-key monitoring prices are null. Those sentences are not a quote. Preview the exact mailboxes, read the returned quote, and follow the paid-run consent rule before connect. Standing auto-enroll is `senders.mailboxes.deliverability_auto_enroll`. Arming is approval-gated because enrollment is a recurring charge bounded by `per_cycle_credit_cap`. Existing mailboxes are not enrolled by arming alone. Detail: [placement.md](placement.md).

## Infrastructure

Domains, DNS, warm-up, managed inboxes, import, OAuth, rotation, and wind-down are in [infrastructure.md](infrastructure.md). Resolve DNS, authorization, and warm-up blockers before any sequence uses the capacity. Size planned daily cold volume as contacts times emails per contact, divided by 42 mailbox-days. Plan about 15 cold emails per inbox per weekday, round up the inbox count, and wait for 14 confirmed warm-up days. Prefer a separate sending domain, mixed providers, and natural addresses.

## Recovery

| What happened | What to do |
| --- | --- |
| Health says unmeasured or stale | Do not treat the inbox as healthy. Read again or run the cheapest missing check. |
| Workspace copy preview and live result disagree, or observations are missing | The test is unmeasured. Read `senders.deliverability.evidence_list`. Do not send a second copy until you know whether the first left. |
| `refused_paused_mailbox` | Leave the pause. Fix the bounce cause and the list. A later healthy probe is not the fix. |
| Purchase or connect quote is refused | Read blockers on the preview. Do not retry live. |
| Warm-up status shows a pause reason | Read `next action` on `warmup_truth`. Resume with `senders.mailboxes.warmup_resume` only after that reason is understood. Resume does not change the subscription. |
| You are unsure a send or purchase landed | Read mailbox list, evidence, and the run. Do not buy or send a duplicate. |

## Worked example

List evidence with `senders.deliverability.evidence_list` and `{"kind":"workspace_copy"}`. If a recent row already answers the question, stop.

Preview with `senders.deliverability.workspace_test_preview`:

```json
{"sender_ids":["aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa"],"receiver_ids":["bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"],"subject":"Q3 rollout note","body_text":"Ada, we cut onboarding from 12 days to 4 for Northwind. Worth a look?"}
```

Read `pair_count`, `proposed_sends` (sender id, receiver id, receiver email), `poll_window_ms`, and `credit_estimate`. If the user approves that exact copy and those ids, call `senders.deliverability.workspace_test_run` with the same params under the paid-run consent rule. Read `evidence_id`, `observations` (`receiver_provider`, `folder`), and `paused_any_mailbox`. Then `senders.deliverability.evidence_compare` with `left_id` of the earlier row and `right_id` of `evidence_id`. Report sample size, provider, folder, and null rates. A missing observation stays missing.

## Decisions

- Cheapest informative check first. Stored evidence before a new send.
- Placement estimates the tested seeds and content. It is not a recipient inbox guarantee.
- Warm-up is a health filter, not proof that production volume is safe.
- Test only the approved sender set. Keep disabled or unauthorized senders blocked.
- Pacing, enrollment caps, and sender binding on a sequence belong to [`outreach`].
- Do not remove a suppression to get a send through.
