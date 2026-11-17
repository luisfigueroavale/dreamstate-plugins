# Assessment

Every read is its own call. A skipped read is unknown.

| Lens | Call | What you record |
| --- | --- | --- |
| Context | `knowledge.brain.context.search`, then `knowledge.brain.context.get` | Offer and ICP confirmed, hypothesis, or missing |
| Outbound | `sequence.analytics.summary` with `window_days` and optional `sequence_id` | `sent`, `replied`, `reply_rate`, `positive_replies`, `meetings`, `active`, `window` |
| Pipeline | `crm.analytics.summary` with `{}` | Stage conversion, win rate, cycle time, score grades |
| Inbox | `unibox.analytics.summary` with the schema's window | `conversations`, `inbound_messages`, `outbound_messages`, `responded_conversations` |
| Content | `social.analytics.summary` with `from`, `to`, and `at_age` | Impressions, engagements, `engagement_rate`, `posts` |
| Sender validity | [`deliverability`] health and readiness | Current, stale, or unmeasured. Do not treat unmeasured as clean. |
| Decisions already owed | `workspace.observability.approvals_list` | Pending items before proposing a new motion |

`reply_rate` is people who replied over people messaged, and is null when nobody was messaged. Win rate is won divided by won plus lost, and is 0 when nothing has closed. Those zeros and nulls are different facts. Say which one the payload is.

## Snapshot

Write this before a recommendation. Every field is a read or the word unknown.

```text
GTM snapshot: <company>, <date>     Context: strong | partial | weak
Goal:            <number and date, or unknown>
Running now:     <sequences, posts, inbox load>
Window:          contacted, replied, reply_rate, positive_replies, meetings, wins
Infrastructure:  known from deliverability, or unknown
Demand on hand:  open deals, responded conversations still waiting
Blind spots:     outreach or CRM this workspace cannot see
```

## Bottleneck order

1. Measurement. If the funnel cannot be read, that is the first finding.
2. Validity. Deliverability, then list quality. A failure here voids every rate below it.
3. Funnel from the top. Mark each stage weak, typical, or strong against this workspace's own history. The binding constraint is the most upstream weak stage.
4. Unworked demand wins over more volume: interested replies with no answer, deals with no recent activity.
5. Capacity last. A motion the team cannot staff is not a motion.

Name the constraint, the runner-up, and the read that would change the call. Sample rows and seeded examples are not demand. Open rates are not evidence. If customers leave right after they win, the constraint is the product, and say so.
