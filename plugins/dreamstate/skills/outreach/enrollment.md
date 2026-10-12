# Enrollment and launch

Default for "enroll this list in a sequence": `sequence.sequences.create_from_worksheet`.

Order of work: preview one real personalized message in chat, dry run, then build the sequence with the worksheet bound as its audience. Building from the worksheet already validates, binds the trigger and stages the rows, so never enroll the same rows again after it. Then publish. Start only when the user explicitly says start.

When the user asked to enroll, do not stop at the draft: publishing is part of enrolling and sends nothing, so build from the worksheet and publish in the same turn, then report that nothing was sent. A published sequence that was never started sends nothing; that is the paused state the user asked for.

Enrollment reads leads straight from the worksheet rows; do not create or import contacts first. A sequence takes one enrollment source: a workbook view, or one workflow, never both. A published sequence bound to a view keeps enrolling rows that later land in or start matching it. Filters on any worksheet column narrow who enrolls; with no filter every row enrolls. When the user said who should get the sequence, filter on the columns that hold it and report how many were held back. Enrollment is free and never sends. Starting is the moment real messages begin, paced over time.

Enrolling from a worksheet binds it as the sequence's trigger. Report the trigger state and the sequence link.

"Enroll in a sequence" means the people worksheet of the workbook the user is looking at, or just built, into a new sequence built for it, even when a company worksheet is open. Use an existing sequence only when the user names it; never report another worksheet's enrollment as the answer.

Publishing and enrolling can happen before any sender is connected. Only starting delivery needs a connected account, and then name exactly which one. Show a preview of real rows in chat as you build; it is not an approval gate. When the user asked to enroll, build and publish in the same turn and let them edit the copy after. Stop for review only when the user asked to approve copy first. There is no send cap to pass: every enrolled lead is worked through, paced by each account's daily limits and sending hours.

After a timeout or an unclear result, read the current enrollment and run state before trying again. Never enroll the same leads twice and never start a second copy of the same sequence.
