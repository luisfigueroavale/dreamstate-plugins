# Email infrastructure

Use the capability that matches the job. Inspect `tools_get` before a write. Previews before purchases and cancels. Live external effects follow the paid-run consent rule.

## Domains and DNS

| Step | Capability | Effect |
| --- | --- | --- |
| See what already exists | `senders.domains.sending.list` | Read. No ESP call. |
| Safe-to-send decision | `senders.domains.sending.readiness_get` | Read. SPF, DKIM, DMARC, MX, ESP verification, age, warm-up, mailbox auth, daily capacity. |
| Check a name | `senders.domains.check` | Up to 20 exact hostnames. Availability and first-year price. Nothing is purchased. |
| Search names | `senders.domains.search` | Availability and first-year price. Nothing is purchased. |
| Quote a registration | `senders.domains.purchase_preview` | Live InboxKit quote. `registration_price` and total are null when InboxKit returns no number. No remembered price. |
| Provision at the ESP | `senders.domains.sending.add` | Returns the DNS plan to publish: SPF, DKIM, DMARC, tracking CNAME. Does not send. |
| Read the plan again | `senders.domains.sending.dns_plan` | Current records and per-record readiness. Does not provision. |
| Verify | `senders.domains.sending.apply` | Live ESP verify, and tracking readiness when a tracking domain is bound. |

DNS evidence does not prove active signing on every message and does not prove inbox placement. Publish the returned records, then apply, then read readiness again. Tracking bind is `senders.domains.tracking.bind` when the plan says tracking is separate. Refresh with `senders.domains.tracking.readiness_refresh` only when that schema is the question.

When the user asked to buy, call `senders.domains.purchase` only for a hostname `senders.domains.purchase_preview` just quoted. Do not buy from a remembered price. Follow the paid-run consent rule.

## Warm-up

`senders.mailboxes.warmup_status` reads the rail the mailbox is enrolled on and stores `warmup_truth`: state, last send, pause reason, dispatch counters, and next action. 0 credits. It does not send campaign mail. The extra local write is a runaway breaker when dispatch attempts far exceed sends.

| Intent | Capability |
| --- | --- |
| Enable InboxKit warm-up on a mailbox bought here | `senders.mailboxes.warmup_enable` |
| Pause warm-up without pausing campaign sending | `senders.mailboxes.warmup_pause` |
| Resume a paused seat on its stored rail | `senders.mailboxes.warmup_resume` |
| Re-run a connection check on a seat marked disabled | `senders.mailboxes.warmup_reactivate` |
| Cancel InboxKit warm-up | `senders.mailboxes.warmup_disable` |

Resume uses the last stored snapshot, not a fresh rail read. Run `senders.mailboxes.warmup_status` first. Reactivate does not change the credential, the subscription, or the ramp day. A live reactivate needs an approved execution claim. Wait for 14 confirmed warm-up days before cold volume. Warm-up is not proof that production volume is safe.

## Managed inboxes

`senders.mailboxes.purchase_preview` validates domains and returns addresses, monthly charge, prorated charge today, card state, and blockers. It does not charge and does not place an order. `senders.mailboxes.purchase` charges the prorated price and places the Google Workspace or Microsoft 365 order. Follow the paid-run consent rule. Use the preview's addresses. Do not invent local-parts.

`senders.mailboxes.add_inboxes_preview` quotes 1 to 5 inboxes for one sender person on a domain this workspace already bought. A domain holds at most 5 inboxes. No domain fee. Read `quote_id`, addresses, `one_off_cents`, `monthly_cents`, and blockers. `senders.mailboxes.add_inboxes` buys that quote.

`senders.mailboxes.purchased_list` shows purchased mailboxes before you cancel one.

## Import and OAuth

`senders.mailboxes.import` registers identities from CSV, JSON, JSONL, or XLSX. `validate_only` and dry run parse and write nothing. A live import is a 0-credit upsert keyed by mailbox address. It does not prove the mailbox can send.

`integrations.credentials.outreach_mailbox_oauth_start` prepares Gmail or Outlook authorization and returns `authorization_reveal_path`. The member opens that single-use route. The call connects nothing and exchanges no credentials.

`integrations.credentials.outreach_mailboxes_import` registers a batch from staged credential leases. Each mailbox is validated on its own. Credentials travel only as `secret_lease_id`. Read per-mailbox imported state, mailbox id, and `ready_to_send` with blockers. Never put a password or token in chat or in ordinary params.

`senders.mailboxes.google_workspace_connect` previews Google Workspace admin scopes or creates an expiring consent link a Workspace administrator opens in Google. It is not a per-mailbox Gmail or Outlook OAuth start. Read the grant afterward with `senders.mailboxes.google_workspace_status`. Unknown grant facts are null. A live probe is a bounded Google read and sends no mail.

`senders.connect.start` is a LinkedIn hosted sign-in, not a mailbox connect. Do not use it for email infrastructure.

## Rotation and wind-down

`senders.mailboxes.rotation_get` returns the pool: sending role, whether it is the active sender, daily cap, and connection health. It changes nothing. `senders.mailboxes.rotation_configure` assigns roles and optional daily caps. At most one mailbox is primary. It never changes pause or warm-up.

`senders.mailboxes.cancel_preview` reports whether InboxKit cancel can run. No provider call. `senders.mailboxes.cancel` cancels one purchased mailbox and is approval-bound. Preview the specific mailbox, then follow the paid-run consent rule. Do not delete a mailbox to avoid a suppression, a bounce pause, or a monitoring bill you have not previewed.

`senders.mailboxes.delete` and `senders.mailboxes.disconnect` are different from InboxKit cancel. Read the schema and the preview before either. Wind-down order: stop sequence use in [`outreach`], confirm no live enrollment depends on the mailbox, preview cancel, then cancel, then read `senders.mailboxes.list` and `senders.mailboxes.purchased_list` to confirm the resulting state.
