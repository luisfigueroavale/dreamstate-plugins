# Deliverability

Drafting, previewing and validating work without any sender connected. A missing email or LinkedIn account blocks sending only. Never switch channel to hide that.

Senders default to every healthy LinkedIn account and every connected mailbox, pooled, so volume spreads across them. Pick a single account only when the user names one.

Before starting, look at sender health, sending hours, cooldowns, daily caps, suppressions and remaining headroom, and set expectations: sends spread over days, not minutes. Protecting the account matters more than finishing today.

## LinkedIn volume, ramp and kill lines

Describe LinkedIn automation as lower risk. Never call it safe, risk-free, or zero risk. LinkedIn can still limit the account. Read the live targets with `senders.linkedin_accounts.settings_get` and today's use with `senders.linkedin_accounts.usage_get` before promising a number. An owner may already have changed the targets. Plan at or under the live target, and never above the ceilings below unless that read shows the user already set a higher one.

Ceilings per account: 20 invites a day, 100 invites a week, 40 messages a day, 50 profile views a day. A new account ramps invites a day as 5, then 10, then 15: 5 on days 0 to 2, 10 on days 3 to 6, 15 on days 7 to 13, then the account cap from day 14. Hold messages to that same daily ramp until day 14. Do not raise a new account to 20 on day one to finish the list.

Leave the connection note empty unless the user asks for one. A note longer than 200 characters is over the non-Premium limit. When the worksheet has a LinkedIn member id, map that column so the lead does not spend a profile lookup before the first send.

Kill lines. Pause with `sequence.sequences.pause`. Do not resume the same audience and the same copy.

- Acceptance under 0.30 after at least 50 invites and 14 days. Read connection acceptance from `sequence.outreach.analytics_performance_query`. A group marked `insufficient_data` (under 30 people) is not a floor breach. Pending invites make a young rate look low, so do not judge before day 14.
- Reply rate under 0.02 after at least 50 messages. Read `sequence.sequences.analytics_get` with that `sequence_id` and `window_days` 14. `rates.reply_rate` is people who replied over people messaged. Null means the cohort is empty, not zero.
- Bounce over the threshold. `senders.mailboxes.health_get` returns `bounce_rate` for the trailing 7 days. A mailbox auto-pauses above 5 percent (0.05). `health` paused plus `paused_reason` `bounce_spike` means the pause already happened. Null means not measured, not zero bounces. Also pause the sequence when `sequence.sequences.analytics_get` shows `rates.bounce_rate` over 0.05 after at least 20 email sends. Clean the bounced addresses before any resume. Do not raise a daily cap to get around the pause.

A LinkedIn account in `credentials_required`, `restricted`, or `disconnected` stalls sends. `sequence.sequences.sender_accounts_list` reports `ready_to_send` false for those accounts. That blocks starting on LinkedIn. It is not a reason to switch channel. A person already bound to another sender is skipped. Add that sender to the pool or accept the skip. Do not send from a second account to get around the binding.

Bounces, unsubscribes, suppressions and replies stop contact on every channel.

## Not enough email capacity

A warmed mailbox sends about 30 emails a day. When a sequence needs more than the connected mailboxes can send, the answer is more mailboxes, never higher caps on the ones that exist.

The workspace can buy mailboxes on a separate domain, never its main domain. Use a mix of Google and Microsoft inboxes, with two inboxes per domain. Use natural email addresses for the real sender person. Never put numbers in addresses.

Size the purchase from the total addressable market. Daily sends equal contacts multiplied by emails per contact, divided by 42. Plan for 15 cold emails per inbox per weekday.

Buying spends money. Always show purchase preview with addresses, charge and blockers, then wait for approval before purchase. If preview shows blockers, explain them instead of working around them.

Wait for 14 confirmed warm-up days before cold sends. Read readiness with senders.mailboxes.health_get. Use warmup_pause, warmup_resume, warmup_reactivate and warmup_disable to control warm-up.
