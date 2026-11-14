# Stops

Unsubscribe, do not contact, bounce, wrong person, decline and a finished conversation all end outreach to that person on every channel. Do not reach them through another sender, another sequence or another channel.

A stop means automation halted, not that a reply went out. After an unclear stop or send result, read current state before retrying.

When a CRM list stays in sync, new members become blocked automatically and members who leave stay blocked.

## Exclusions

Workspace exclusions are named rules over people or companies. The defaults are Internal team, Schools, Customers, Open deals, Booked a meeting, Existing conversations, and LinkedIn connections. Competitors is set up by the user.

Each exclusion has channels. Saving one blocks new enrollments and stops active enrollments at their next send. Existing conversations and LinkedIn connections are checked at enrollment only.

To answer "why was this person skipped", call `senders.suppressions.exclusions_explain`. To change one, use `senders.suppressions.exclusions_update`. Never edit sequence copy to work around an exclusion.

Unsubscribes and bounces are always on and are restored only through consent restore.
