# Enrollment and launch

Default for "enroll this list in a sequence": `sequence.sequences.create_from_worksheet`.

`sequence.sequences.create_from_worksheet` already validates, binds the trigger and stages the rows, so never enroll the same rows again after it. Publish when the returned readiness checks pass. Start only when the user explicitly says start.

## Launch ladder

Run these in order. A later step does not replace an earlier one. Publishing and enrolling never send.

1. Validate. Call `sequence.sequences.validate` with `definition` (the same name and compose-shaped steps) and `worksheet_id`. `valid` must be true. Fix every issue in `issues` and validate again. Warnings such as `missing_fallback_template` or `linkedin_dm_without_connection_request` are fixed before publish. `sequence.sequences.create_from_worksheet` returns the same `validation` object. Do not treat a chat draft as validation.
2. Preview one real message. After the draft exists, call `sequence.sequences.copy_preview` with `sequence_id` and one `record_ids` entry, a real worksheet row. Show that rendered body in chat, with the personalized slots called out. `limit` defaults to 3 and cannot exceed 10. Use one row for this step. `no_staged_or_enrolled_leads` means nothing is staged yet, so stage first, then preview again.
3. Dry run. Dry run is the tools_run mode. Put `dry_run` inside params only when the fetched schema declares that field. `sequence.sequences.start` does not. Its params are only `{ "sequence_id": "<id>" }`. Call `sequence.sequences.launch` in dry run, then after publish call `sequence.sequences.start` in dry run. Dry run writes no send. Read blockers, capacity, and suppressions. A disconnected sender blocks starting only.
4. Small first cohort. `sequence.sequences.publish` when readiness passes. It sends nothing. Live `sequence.sequences.start` only when the user says start, under the paid-run consent rule. Keep day one to the qualified `enrollment_filter` and a low `max_new_enrollments_per_day`. Do not open the whole worksheet on the first live start. A live email, WhatsApp, or CRM-task start follows the send-ceiling rule in [SKILL.md](SKILL.md) State.
5. Ramp. Leave it inside the account ceilings in [deliverability-and-caps.md](deliverability-and-caps.md). Read `sequence.sequences.analytics_get` before any volume increase. Raise one limit at a time.
6. Full. Enroll the rest of the qualified rows only after the kill lines in [deliverability-and-caps.md](deliverability-and-caps.md) still hold. Full is a later start, not the first call.

`sequence.sequences.create_from_worksheet` stages the view in one request. It does not page on `has_more`. `max_new_enrollments_per_day` paces who is admitted each day. It does not remove people already staged. When `enrollment.created` is 0, read `enrollment.skipped_reasons` and call `senders.suppressions.exclusions_explain` before changing the filter. Name every unresolved row, not only the count. Do not widen the filter to make a count nonzero. A later add uses `sequence.sequences.enroll_selection`, whose `max_new_enrollments_per_day` is at most 500.

When the user asked to enroll, build and stage the requested audience, then publish when readiness passes. A disconnected sender or missing copy evidence can leave a useful draft with an exact repair; report that state honestly. A published sequence that was never started sends nothing. Published-but-unstarted and manually paused are distinct states.

Enrollment reads leads straight from the worksheet rows; do not create or import contacts first. A rendered preview and a launch rehearsal use those row values even when no CRM person exists yet. On live enrollment, the system matches each usable row identity to one existing person or creates and binds the missing person automatically. A sequence can have ordered worksheet audiences, each with its own enrollment column filter, identity mapping, priority and cap, or a workflow source. Keep personalization provenance on its selected source; do not silently substitute another audience when a source is archived. A published sequence can enroll worksheet rows that later match its enrollment filter. Filters on any worksheet column narrow who enrolls; with no filter every row enrolls. When the user said who should get the sequence, filter on the columns that hold it and report how many were held back. Enrollment is free and never sends. Starting is the moment real messages begin, paced over time.

When a lead is skipped, read Exclusions in [stop-and-suppression.md](stop-and-suppression.md). Call `senders.suppressions.exclusions_explain` before changing enrollment filters or sequence copy.

Do not turn a pending binding preview into a request to create CRM contacts. Report the returned blocker precisely: a row without usable channel identity, an ambiguous/conflicting identity, a permission refusal, or the named launch/readiness issue. The CRM-only operation that creates records for unbound rows remains available for intentional CRM cleanup or import, never as an outreach prerequisite.

Social account post and AI answer worksheets hold content rows, never an audience or subject. To reach authors, send unique author URLs to a people worksheet first.

Enrolling from a worksheet binds it as the sequence's trigger. Report the trigger state and the sequence link.

"Enroll in a sequence" means the people worksheet of the workbook the user is looking at, or just built, into a new sequence built for it, even when a company worksheet is open. Use an existing sequence only when the user names it; never report another worksheet's enrollment as the answer.

Draft creation and staging can happen when no rows match the enrollment filter or the sender is disconnected. Keep the worksheet and enrollment filter unchanged; never widen the audience to make a count nonzero. Use top-level `identity_mapping` in `sequence.sequences.create_from_worksheet`, for example `{linkedin_url_column_key:"person.linkedin_url"}`; the service saves it on the audience source. Keys name worksheet columns or declared nested outputs. Do not pass the removed `linkedin_profile_column_id` override.

For an identity failure, inspect the current strict input schema and submit the exact returned identity_mapping repair on that audience without changing its worksheet or enrollment filter. Read back the saved mapping and render the intended recipient before reporting the repair complete.

Publishing and starting enforce their returned readiness checks; connect the named channel sender or repair the exact missing evidence. Publishing and enrolling never send. Show previews of rich, thin and missing-evidence rows as you build, labeling fallback copy or a hold explicitly. If a row sample is incomplete, report its sample size instead of calling it the full audience. Preserve configured sequence, audience and sender limits. Start only on the user's explicit instruction and the canonical paid-run consent rule.

After a timeout or an unclear result, read the current enrollment and run state before trying again. Never enroll the same leads twice and never start a second copy of the same sequence. An outcome you cannot see is not a failure to retry. Follow the recovery ladder in [stop-and-suppression.md](stop-and-suppression.md).

## Worked example

Qualified people worksheet, LinkedIn URLs, a public hiring column `matched_job`, and the user asked to start a first cohort of 15 after seeing one real message. Nothing below sends until the live start.

1. Validate the definition. `sequence.sequences.validate` params:

```json
{
  "worksheet_id": "33333333-3333-4333-8333-333333333333",
  "definition": {
    "name": "Hiring founders",
    "steps": [
      {
        "kind": "email",
        "fields": {
          "channel": "email",
          "subject_template": "{{company}} role",
          "body_template": "Hi {{first_name}}, saw {{company}} is hiring an {{matched_job}}. Worth a look at how we help that hire?",
          "fallback_template": "Hi, saw the open role. Worth a look?"
        }
      }
    ]
  }
}
```

Read `valid` and `issues`. Fix and call again until `valid` is true. `{{matched_job}}` is a public hiring signal. Do not put a profile view or a site visit in the body.

2. Create and stage. `sequence.sequences.create_from_worksheet` dry run first, then the same params live. Dry run writes nothing and can return `matching_blueprints`. Use a blueprint when one matches. Otherwise live params:

```json
{
  "worksheet_id": "33333333-3333-4333-8333-333333333333",
  "name": "Hiring founders",
  "identity_mapping": { "linkedin_url_column_key": "person.linkedin_url" },
  "context_document_ids": ["11111111-1111-4111-8111-111111111111"],
  "context_revision_ids": ["22222222-2222-4222-8222-222222222222"],
  "max_new_enrollments_per_day": 15,
  "enrollment_filter": { "kind": "predicate", "field": "col:fit_score", "op": "gte", "value": 80 },
  "steps": [
    {
      "kind": "email",
      "fields": {
        "channel": "email",
        "subject_template": "{{company}} role",
        "body_template": "Hi {{first_name}}, saw {{company}} is hiring an {{matched_job}}. Worth a look at how we help that hire?",
        "fallback_template": "Hi, saw the open role. Worth a look?"
      }
    }
  ]
}
```

Read back `sequence_id`, `sequence_version_id`, `validation.valid`, `context.document_ids`, `enrollment.created`, `enrollment.skipped`, and `enrollment.skipped_reasons`. Do not enroll those rows again.

3. Preview one real message. `sequence.sequences.copy_preview` params `{ "sequence_id": "<sequence_id>", "record_ids": ["<one worksheet row id>"] }`. Show `resolved body` for that row. If a variable is unresolved, stop and fix the copy.

4. Publish with `sequence.sequences.publish` params `{ "sequence_id": "<sequence_id>", "sequence_version_id": "<sequence_version_id>" }`. Status `published` still sends nothing.

5. Dry run start. tools_run mode dry run. `sequence.sequences.start` params are only `{ "sequence_id": "<sequence_id>" }`. Do not add `dry_run` to those params. The schema is strict and rejects it. Read blockers. Do not go live while a blocker remains.

6. Live start for the first cohort only after the user says start, under the paid-run consent rule. Same `sequence.sequences.start` params, live. The daily cap of 15 is the first cohort. It is not the whole worksheet.

7. Read back. `workflow.automations.get` for the saved steps, filter, and context ids. `sequence.sequences.events_list` params `{ "sequence_id": "<sequence_id>", "kinds": ["sent", "failed", "deferred"], "limit": 50 }`. If the start receipt has a `run_id` and `persisted` true, `runs.get` that `run_id`. `persisted` false has no api_runs row, and `runs.get` returns `not_found`. Report enrolled, skipped, and that delivery is paced. Do not claim a send the events page does not show.
