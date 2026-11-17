# Stops

Unsubscribe, do not contact, bounce, wrong person, decline and a finished conversation all end outreach to that person on every channel. Do not reach them through another sender, another sequence or another channel.

A stop means automation halted, not that a reply went out.

A reply of any kind ends this enrollment on every channel. That stop is this sequence. It does not by itself prove a workspace-wide block. Read `senders.suppressions.rules_get` before the first live start and say which automatic rules are on. Do not turn an unsubscribe or a bounce rule off. Existing conversations and LinkedIn connections are checked at enrollment only, so a later reply is what stops the running enrollment. A WhatsApp reply stops that WhatsApp enrollment. It does not block the person's email or LinkedIn. To block a phone, an email, or a domain for later sequences, call `senders.suppressions.add`. `kind` is `email`, `domain`, `company`, `phone`, `linkedin`, `x`, or `instagram`. `reason` is the literal `manual`. A `domain` value blocks that whole domain. Follow the fetched schema. Explain a skip with `senders.suppressions.exclusions_explain` before changing copy or filters.

## Recovery when the outcome is unknown

Never resend blind. An unknown outcome is not a failed send. Do not mint a new request id and fire the same message again.

1. Read the run. When the receipt has a `run_id` and `persisted` true, call `runs.get`. `persisted` false has no api_runs row, and `runs.get` returns `not_found`. Then read `sequence.sequences.events_list` for that sequence with `kinds` `sent`, `failed`, and `deferred`.
2. Read the conversation with `unibox.inbox.conversation_get` before any retry. A conversation status is not proof of delivery. If the message is already in the thread, stop.
3. Ambiguous LinkedIn invite or message: `sequence.sequences.pause` first. Then `sequence.sequences.linkedin_reconcile` with `sequence_id`, `action_id`, and `dry_run` true. That schema declares `dry_run`. `approved` true only records an exact provider match or a refusal. The call never resends (`resend` is false). Send again only when `outcome` is `refused`. `matched` means it already went out.
4. Window holds: on a paused sequence, `sequence.sequences.retry_deferred` in tools_run dry run returns `approval_token` and writes nothing. Its params are `sequence_id` and `limit` only. Do not put `dry_run` in those params. The live call adds the same `approval_token` and the same `limit`. Eligible reasons are `outside_account_send_window` and `email_send_window_hold`. Then `sequence.sequences.resume` is what allows delivery. The retry call itself never sends. `sequence_repair_requires_paused` means pause first. `sequence_retry_deferred_approval_token_mismatch` means dry run again.
5. Provider capacity denials: `sequence.sequences.recover_capacity` while paused, then `sequence.sequences.recover_capacity_apply` with that exact `approval_token`. Apply never resends. A token mismatch means preview again.

Each of these previews first and, except the reads, runs only on a paused sequence.

When a CRM list stays in sync, new members become blocked automatically and members who leave stay blocked.

## Exclusions

Workspace exclusions are named rules over people or companies. The defaults are Internal team, Schools, Customers, Open deals, Booked a meeting, Existing conversations, and LinkedIn connections. Competitors is set up by the user.

Each exclusion has channels. Saving one blocks new enrollments and stops active enrollments at their next send. Existing conversations and LinkedIn connections are checked at enrollment only.

To answer "why was this person skipped", call `senders.suppressions.exclusions_explain`. To change one, use `senders.suppressions.exclusions_update`. Never edit sequence copy to work around an exclusion.

Workspace exclusions apply to every sequence by default. A sequence turns one off for itself by listing that rule's id in `disabled_exclusion_ids` on the `patch` of `workflow.automations.update` (stored as `settings.disabled_exclusion_ids`). Take the id from `senders.suppressions.exclusions_list`. A rule not in the list still applies, including a rule created later. A customer upsell or expansion sequence turns off Customers. A sequence meant for existing LinkedIn connections turns off LinkedIn connections. The list replaces the previous one, and an empty list turns every exclusion back on. Turn one off only when the user's goal clearly calls for it, and say that in the reply.

Unsubscribed, bounced, and manual list entries can never be turned off. Unsubscribes and bounces are always on and are restored only through consent restore.
