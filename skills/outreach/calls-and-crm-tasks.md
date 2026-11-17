# Call steps and CRM tasks

Two step kinds hand work to a person. Neither one messages the lead by itself. Read `sequence.sequences.step_options` first. `crm_task` is not in that list. Authoring it fails. The owner removed the standalone CRM task step on 2026-11-16. `manual_review`, `wait`, `wait_for_connection`, `fetch_profile`, and `withdraw_invite` are also not authorable.

## When to use a call

Use `kind` `call` when a person must phone the lead and the journey should wait for the outcome. The step queues one call task. It sends nothing. `talking_points` is guidance for the rep, up to 4000 characters, and is never sent. `task_title` is the headline on that task, up to 200 characters. Both accept sequence variables.

`next_step_id` is the picked-up branch, when the contact connects. `no_answer_step_id` is the branch after `voicemail`, `no_answer`, `busy`, `failed`, `skipped`, or timeout. Null ends that arm. Do not write `on_timeout` or `on_connected`. Those are legacy and the no-answer branch replaces them.

`timeout_days` is 1 to 30 and defaults to 3. It counts from when the task is queued. The dispositions a later condition can read on `call_disposition` are `connected`, `no_answer`, `voicemail`, and `wrong_number`.

`assignee` is optional:

- `{ "source": "sequence_sender" }` gives the call to the sender on this enrollment.
- `{ "source": "members", "user_ids": ["<uuid>"], "rotation": "round_robin" }` rotates across 1 to 20 workspace members.
- `{ "source": "record_owner" }` uses the linked record owner.
- `{ "source": "row", "column_key": "<column>" }` reads a member id or email from the enrolled row.

Omit `assignee` only when the workspace default caller is acceptable. Do not invent a user id. Resolve members before putting them in `user_ids`.

A call is not a task whose `channel` is `call`. That task opens a to-do. It does not park the enrollment on a dialer disposition.

## When to use a task

Use `kind` `action` and `action` `create_task` when a person in the workspace needs a to-do, or when that to-do should also create one HubSpot task. The step always creates the Dreamstate task. HubSpot is optional and only runs when `payload.crm` is set.

`payload` requires `title` (1 to 300 characters, row tokens allowed). Optional fields:

- `description`, up to 10000 characters, or null.
- `priority`: `low`, `normal`, or `high`.
- `assignee_user_id`: a workspace member uuid, or null. Omitted, the native task defaults to the sequence owner.
- `assignee`: `{ "source": "row", "column_key" }` or `{ "source": "record_owner" }`. A dynamic assignee requires `fallback_assignee_user_id`. Without it, launch returns `Create task needs a fallback assignee.`
- `due_in_minutes`: a positive integer up to 525600, or null for no due date. 1440 is one day.
- `channel`: `email`, `linkedin`, `x`, `whatsapp`, `instagram`, or `call`. The task opens the composer on that channel. Omit it for a general to-do.
- `message`: drafted composer text, up to 4000 characters. Omit for no draft.

`payload.crm` is the HubSpot task. It requires `enabled` true, `provider` `hubspot`, `property_mappings`, and `associate`. `property_mappings` requires:

- `hs_timestamp`: `{ "source": "relative_time", "offset_minutes": <integer> }`.
- `hs_task_subject`: `{ "source": "template", "template": "<text>" }`, at most 255 characters.

Optional mappings: `hs_task_body` as a template up to 65536 characters, `hs_task_type` as `{ "source": "literal", "value": "CALL" | "EMAIL" | "TODO" }`, `hs_task_priority` as a literal or `{ "source": "row", "column_key" }`, and `hubspot_owner_id` the same way. `associate.contact_by` is `email` or `none`. `associate.company_by` is `domain` or `none`. LinkedIn URL matching is unsupported. `on_missing` is `skip` or `fail` and defaults to `skip`.

When `crm` is present, launch requires a usable owner. A missing owner, a non-numeric literal, or an empty row key returns `Create task CRM owner is required.` Read `integrations.hubspot_owners_list` and store the numeric id as `{ "source": "literal", "value": "<digits>" }`. A revoked HubSpot grant fails that list with `hubspot_reconnect_required`. Missing owners scope fails with `feature_unavailable` and reason `hubspot_scope_not_granted`. A missing connection fails with `connection_required`.

A HubSpot task is an external write. A live start that will create one follows the paid-run consent rule and the ceiling SKILL.md states for that family. Preview the step with `workflow.automations.lint` before that start. `skip` on a missing HubSpot record continues the enrollment. `fail` stops that lead's task write. Do not switch to `fail` to force a match.

## Worked example

After the first email, a rep calls. On pickup the sequence stops. On no answer it creates a HubSpot call task due the next day, owned by the connected HubSpot user `42`.

1. `sequence.sequences.step_options` with `{}`. Confirm `call` and `create_task` are present and `crm_task` is absent. If `crm_task` is absent, do not send it.
2. `workflow.automations.node_inspect` for the call subtype and for `create_task`. Use only returned fields.
3. `integrations.hubspot_owners_list` with `{}`. Use a returned numeric id. Do not invent `42` if the list does not contain it.
4. `workflow.automations.node_add` the task first, `kind` `sequence`, so its id exists. `node_add` documents `branch` `"false"` for a condition's false arm only. Do not invent a branch name for the call. Point the call at the task with `no_answer_step_id`.

```json
{
  "id": "task-1",
  "kind": "action",
  "action": "create_task",
  "payload": {
    "title": "Call {{first_name}} about the pilot",
    "priority": "high",
    "due_in_minutes": 1440,
    "crm": {
      "enabled": true,
      "provider": "hubspot",
      "property_mappings": {
        "hs_timestamp": { "source": "relative_time", "offset_minutes": 1440 },
        "hs_task_subject": { "source": "template", "template": "Call {{first_name}}" },
        "hs_task_type": { "source": "literal", "value": "CALL" },
        "hubspot_owner_id": { "source": "literal", "value": "42" }
      },
      "associate": { "contact_by": "email", "company_by": "domain" },
      "on_missing": "skip"
    }
  }
}
```

The literal `"42"` is valid only when step 3 returned that id. Then add the call after the email, with `after_node_id` set to the email step id, and patch it if the add did not keep both branch ids:

```json
{
  "id": "call-1",
  "kind": "call",
  "task_title": "Call {{first_name}}",
  "talking_points": "Ask whether the pilot timeline still holds. Do not pitch a second product.",
  "assignee": { "source": "sequence_sender" },
  "timeout_days": 3,
  "next_step_id": null,
  "no_answer_step_id": "task-1"
}
```

5. Readback: `workflow.automations.lint` is clean, and `workflow.automations.get` shows `call-1` with `no_answer_step_id` `task-1` and `next_step_id` null, and `task-1` with `crm.provider` `hubspot`. No step has action `crm_task`. `Create task CRM owner is required.` means the owner was missing or not all digits. Replace it with the id from `integrations.hubspot_owners_list` and lint again. Nothing has sent.
