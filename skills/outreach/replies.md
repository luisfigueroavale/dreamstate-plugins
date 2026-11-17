# Replies

A reply, bounce, unsubscribe, suppression or decline ends the sequence for that person. Read the whole current thread before drafting an answer, and answer what they said instead of restarting the pitch.

WhatsApp and Instagram replies stop enrollment and appear in Unibox. Read that thread there before replying.

Instagram follow and like steps are actions, not replies. A follow of a private account sends a request, and a like targets the most recent accessible post. Both share the daily action budget with Instagram DMs.

A conversation status is not proof a message was delivered. After an unclear send result, do not resend blind.

1. Read the run with `runs.get` when the receipt has a `run_id` and `persisted` true. `persisted` false returns `not_found` from `runs.get`. Read `sequence.sequences.events_list` for `sent`, `failed`, and `deferred`.
2. Read the thread with `unibox.inbox.conversation_get`. If the message is already there, stop. Do not mint a new request id and send it again.
3. If the provider outcome is still unknown, pause with `sequence.sequences.pause`, then reconcile. For a LinkedIn invite or message, `sequence.sequences.linkedin_reconcile` params are `sequence_id`, `action_id`, and `dry_run` true, then `approved` true only to record a match or a refusal. It never resends. Send again only when `outcome` is `refused`.
4. A window hold uses `sequence.sequences.retry_deferred` on the paused sequence. Dry run is the tools_run mode. Params are `sequence_id` and `limit`, then the live call adds `approval_token`. A capacity denial uses `sequence.sequences.recover_capacity`, then `sequence.sequences.recover_capacity_apply` with that token. Neither call sends. `sequence.sequences.resume` is the later step that allows delivery.

The same ladder is in [stop-and-suppression.md](stop-and-suppression.md). Reply in the open thread. Do not add a sequence step to answer it.
