# Two-sided qualification

Qualified means both fit gates passed. It does not mean approved to send.

## Rubric

Score the company and the person separately. Put both sides on one `column.ai_fit_score` column. That column is the fit score. Do not add a second generic AI column to grade the same fit. Its rubric lists every criterion as Required or Nice to have.

- Company criteria: business model, industry or use case, size, region. The person's own location decides region. Company headquarters is the fallback. A place outside the ideal customer regions fails company fit.
- Person criteria: current role and remit, and whether they own the problem. A title alone is not purchasing authority.
- Both sides are Required. A Required fail scores 0. Missing evidence is unsure, never a pass.
- Hard exclusions override any score: competitor, existing customer, the author's team, and an open conversation. Set the row to fail and name the exclusion.
- Engagement, its date, and the signal type stay out of the score. They sort rows only after both Required sides pass. A comment, a follow, a profile view, or a funding headline never lifts a failed fit.

`column.ai_fit_score` returns a numeric `score` from 0 to 100. Views and enrollment read `fit_score_value` (the `score` field), not a made-up pair of subscores. If you need the two sides visible, require the column reason to name the company result and the person result in words. Do not promise fields the column does not return.

The CRM grade is a different read. `crm.records.score_explain` returns `score.grade` and `score.signal_level` separately. Use the worksheet criteria to decide who enters a warm sequence. Use the CRM grade when the person already has a record. Do not average them.

## Calibrate, then batch

1. Dry-run the fit column on one row and read the rendered prompt and the estimate.
2. Run four rows: a clear fit, a wrong company, a wrong person, and a row missing the evidence a Required criterion needs. Adjust the rubric until all four are right. Unsure stays unsure.
3. Quote the next batch under the paid-run consent rule. Run later columns only where the Required criteria already pass.

A changed rubric is a new column. Do not pretend old scores were recalculated.

## Exclude before enrolling

Drop do-not-contact, customers, people already in another sequence, and anyone `sequence.sequences.contacts_list` shows as contacted on the sequence you would use. One person per company in these warm plays, so a company hears from the sender once.

Enrollment uses the qualified view's `view_id` on `sequence.sequences.enroll_selection`. `visibility.linkedin.intent_autoenroll` has no fit gate. Never use it as the way people enter a sequence.
