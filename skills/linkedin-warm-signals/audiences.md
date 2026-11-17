# Audience plays

Run one audience at a time. Fit comes from [qualification.md](qualification.md). The sequence itself is [`outreach`]. These caps are the play's defaults. The account's own daily limit and sending hours still apply. A warm sequence is three steps or fewer, with at most one nudge 5 to 7 days after the first message.

## Post engagers

Use when the user posts and wants the people who commented or reacted.

- Capture the user's own posts with `source.engaged_with_account`. A competitor or peer page is `source.engaged_with_company` on its own worksheet.
- Enroll only rows that pass every Required company criterion and every Required person criterion.
- A public comment may be quoted. A reaction alone is private: do not say you saw it. You may mention the public post.
- Lead-magnet commenters are owed the resource, not a pitch. Leave them out.
- Kill line: if the replies are mostly people who fail fit, tighten the Required criteria before you change the copy. If acceptance collapses, pause enrollment and re-read the rubric.
- Scale by capturing more of the user's own posts, or by a paid watcher of a few relevant public profiles (`tables.profile_watcher.preview` first). Never scale by lowering the fit gate.

## Profile viewers and followers

Use when the user wants named viewers or people who follow without being connected. Viewers go first.

- Capture viewers with `source.linkedin_profile_viewers` or `visibility.linkedin.viewers_import`. Capture the account's followers with `source.linkedin_followers` or `visibility.linkedin.followers_import`. Someone else's company page is `tables.followers.order`, a paid order, not the account import.
- Followers often have no follow date. Say that recency is unknown.
- Default cap: at most 10 new people a day on the sequence.
- The first line leads with relevance. Never say they viewed the profile or followed.
- Kill line: if replies or accepts fall and the follower rows are the drag, stop enrolling followers and keep viewers. Re-score only new rows.

## New connections

Use when the user wants a follow-up with people who connected recently. They are already first degree, so this play sends messages only and does not spend an invite.

- Capture with `source.linkedin_new_connection`. Read `visibility.linkedin.network_capture_status` until new rows land. A first walk still filling is not an empty market.
- Default cap: at most 10 new people a day.
- Check for an existing thread before you enroll. A hit belongs to the reply skill, not a new sequence.
- Kill line: read the first replies before you make this daily. If they are mostly non-buyers, tighten the rubric. Do not add a second nudge to compensate.

## Existing network

Use when the user says to work the network or reactivate connections. This play is hand-written. Do not bulk-enroll the network into a template.

- Capture with `source.linkedin_connections` through `visibility.linkedin.network_capture_setup`. Pause with `visibility.linkedin.network_capture_pause`. Setup and pause do not call LinkedIn.
- Expect a short list after the fit gate. One message per company, to the buyer or to the person who engages in public.
- Draft about 5 for a new sender, then batches of 10. No automated follow-up.
- The opener may use a public fact (a hire, a round, a public post). Leave out layoffs and anything private.
- Kill line: if a reply is an apology for an automated note, stop the batch. Scale by the next hand-written batch, not by switching on a template.

## Shared kill switch

Pause the sequence in [`outreach`] and pause the source or watcher. Captured rows stay. Do not delete them to make the list look clean.
