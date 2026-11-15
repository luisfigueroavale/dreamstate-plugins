# Stops

Unsubscribe, do not contact, bounce, wrong person, decline and a finished conversation all end outreach to that person on every channel. Do not reach them through another sender, another sequence or another channel.

A stop means automation halted, not that a reply went out. After an unclear stop or send result, read current state before retrying.

When a CRM list stays in sync, new members become blocked automatically and members who leave stay blocked.

## Exclusions

Workspace exclusions are named rules over people or companies. The defaults are Internal team, Schools, Customers, Open deals, Booked a meeting, Existing conversations, and LinkedIn connections. Competitors is set up by the user.

Each exclusion has channels. Saving one blocks new enrollments and stops active enrollments at their next send. Existing conversations and LinkedIn connections are checked at enrollment only.

To answer "why was this person skipped", call `senders.suppressions.exclusions_explain`. To change one, use `senders.suppressions.exclusions_update`. Never edit sequence copy to work around an exclusion.

Workspace exclusions apply to every sequence by default. A sequence turns one off for itself by listing that rule's id in `disabled_exclusion_ids` on the `patch` of `workflow.automations.update` (stored as `settings.disabled_exclusion_ids`). Take the id from `senders.suppressions.exclusions_list`. A rule not in the list still applies, including a rule created later. A customer upsell or expansion sequence turns off Customers. A sequence meant for existing LinkedIn connections turns off LinkedIn connections. The list replaces the previous one, and an empty list turns every exclusion back on. Turn one off only when the user's goal clearly calls for it, and say that in the reply.

Unsubscribed, bounced, and manual list entries can never be turned off. Unsubscribes and bounces are always on and are restored only through consent restore.
