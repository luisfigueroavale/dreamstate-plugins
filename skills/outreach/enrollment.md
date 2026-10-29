# Enrollment and launch

Default for "enroll this list in a sequence": `sequence.sequences.create_from_worksheet`.

Order of work: preview one real personalized message in chat, dry run, then build the sequence with the worksheet bound as its audience. Building from the worksheet already validates, binds the trigger and stages the rows, so never enroll the same rows again after it. Publish when the returned readiness checks pass. Start only when the user explicitly says start.

When the user asked to enroll, build and stage the requested audience, then publish when readiness passes. A disconnected sender or missing copy evidence can leave a useful draft with an exact repair; report that state honestly. A published sequence that was never started sends nothing. Published-but-unstarted and manually paused are distinct states.

Enrollment reads leads straight from the worksheet rows; do not create or import contacts first. A rendered preview and a launch rehearsal use those row values even when no CRM person exists yet. On live enrollment, the system matches each usable row identity to one existing person or creates and binds the missing person automatically. A sequence can have ordered worksheet audiences, each with its own saved view, filter, identity mapping, priority and cap, or a workflow source. Keep personalization provenance on its selected source; do not silently substitute another audience when a source is archived. A published sequence bound to a view keeps enrolling rows that later land in or start matching it. Filters on any worksheet column narrow who enrolls; with no filter every row enrolls. When the user said who should get the sequence, filter on the columns that hold it and report how many were held back. Enrollment is free and never sends. Starting is the moment real messages begin, paced over time.

Do not turn a pending binding preview into a request to create CRM contacts. Report the returned blocker precisely: a row without usable channel identity, an ambiguous/conflicting identity, a permission refusal, or the named launch/readiness issue. The CRM-only operation that creates records for unbound rows remains available for intentional CRM cleanup or import, never as an outreach prerequisite.

Social account post and AI answer worksheets hold content rows, never an audience or subject. To reach authors, send unique author URLs to a people worksheet first.

Enrolling from a worksheet binds it as the sequence's trigger. Report the trigger state and the sequence link.

"Enroll in a sequence" means the people worksheet of the workbook the user is looking at, or just built, into a new sequence built for it, even when a company worksheet is open. Use an existing sequence only when the user names it; never report another worksheet's enrollment as the answer.

Draft creation and staging can happen with an empty qualified view or disconnected sender. Keep the saved view and filters unchanged; never widen the audience to make a count nonzero. Use top-level `identity_mapping` in `sequence.sequences.create_from_worksheet`, for example `{linkedin_url_column_key:"person.linkedin_url"}`; the service saves it on the audience source. Keys name worksheet columns or declared nested outputs. Do not pass the removed `linkedin_profile_column_id` override.

For an identity failure, inspect the current strict input schema and submit the exact returned identity_mapping repair on that audience without changing its worksheet, saved view or filter. Read back the saved mapping and render the intended recipient before reporting the repair complete.

Publishing and starting enforce their returned readiness checks; connect the named channel sender or repair the exact missing evidence. Publishing and enrolling never send. Show previews of rich, thin and missing-evidence rows as you build, labeling fallback copy or a hold explicitly. If a row sample is incomplete, report its sample size instead of calling it the full audience. Preserve configured sequence, audience and sender limits. Start only on the user's explicit instruction and the canonical paid-run consent rule.

After a timeout or an unclear result, read the current enrollment and run state before trying again. Never enroll the same leads twice and never start a second copy of the same sequence.
