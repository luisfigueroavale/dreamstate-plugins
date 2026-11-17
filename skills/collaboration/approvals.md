# Approval requests, decisions, and gates

## What can block a live path

| Gate | What arms it | What opens it | What this catalog does not do |
| --- | --- | --- | --- |
| Post routing rule | `workspace.approval_rule_create` | Matching approvers decide through content approval | Does not block sequence launch |
| Content review | `social.content.approval.submit` | `social.content.approval.decide` with `decision` `approved` on that revision | `changes_requested` does not open it |
| Publish authorization row | `social.content.approval.authorize_publish` | The caller self-approves that revision | Not a second-party decision |
| Workflow run pause | A workflow leaves an approval row | `workflow.approvals.decide` with the reviewed `plan_hash` | Not a content decision |
| Standing policy | `workspace.standing_approval_policy_set` | The operator's own policy | Not a named teammate's sign-off |
| Paid or external effect | The paid-run consent rule on that capability | The same quote, ceiling, and approval the rule requires | A comment never satisfies it |

There is no sequence `launch` gate and no worksheet `signoff` gate in the catalog. Do not invent `approval_required` from a thread. If a live sequence call is refused, read the refusal in [`outreach`] and the paid-run consent rule.

`workspace.observability.approvals_list` lists pending workflow approvals, publishing approvals, message reviews, inbox reply drafts, and collaboration requests. Each item includes the command that resolves it and a web link. A source that cannot be read is null, not empty.

## Content approval

`social.content.approval.review_queue` takes `{}` and returns `needs_my_review`, `my_submitted`, and `recently_decided`.

Submit params:

- `artifact_id`
- `expected_revision` equal to the artifact's current revision, or 409 `stale_revision`
- `approvers`: `user_id`, `routed_by` (`manual`, `rule`, or `default`), optional `rule_id`, `stage` (`agency_review` or `client_review`, default `client_review`). `round` is ignored. The server keeps a still-pending request or opens a new round when the person already decided or an edit invalidated the approval.
- `needed_by` as an ISO 8601 timestamp with offset
- optional `note` up to 2,000 characters

Each approver must hold publishing role `publisher` or `manager`, or the whole submission is rejected.

Decide params:

- `approval_ids` (1 to 50), the caller must be the designated approver or the call is 403
- `expected_revision` bound at submission
- `decision`: `approved` or `changes_requested`
- `reason` required for `changes_requested`, optional otherwise

The same terminal decision again returns receipts with `replayed` true. A conflicting terminal decision is 409 `approval_already_decided`. A failed notification email does not undo the decision.

`social.content.approval.authorize_publish` takes `artifact_id`, `expected_revision`, and `needed_by`. It writes a self-approved row at stage `client_review` in a fresh round, caller as requester and approver. Use it only when this caller is supposed to authorize publication. Do not use it to jump a rule that named other approvers.

## Workflow approvals

`workflow.approvals.list` filters by optional `workflow_id`, optional `run_id`, `status` (`pending` default, or `approved`, `rejected`, `expired`, `all`), and `limit`.

`workflow.approvals.decide` takes `approval_id`, `decision` `approve` or `reject`, `plan_hash` matching `sha256:` plus 64 hex characters, and `preview` (default true). Preview first. A live decision uses the hash from that preview. This `approve` value belongs to workflow approvals. Content decisions use `approved`.

## Rules

`workspace.approval_rules_list` is paged. Create with `name`, `conditions` (`match` array of `field`, `op`, `value`, plus `operator` `AND` or `OR`), `approver_user_ids`, `require_all` (default true), `enabled` (default true), and `priority` (default 100, lower first). Approver ids must be members with publishing role `publisher` or `manager`. Others are `invalid_request`.

`workspace.approval_rule_archive` takes `rule_id`. The row stays with `archived_at` set, leaves the list, and stops routing. In-flight items are not rewritten. Read the list before creating a duplicate rule.

## Standing policy

`workspace.standing_approval_policy_get` takes `{}` and returns `policy` with `capability_ids`, `effect_classes` (`none`, `internal_reversible`, `external`, `destructive`), and `credit_floor`. `workspace.standing_approval_policy_set` replaces the fields you pass. `credit_floor` null clears the floor. `workspace.standing_approval_policy_clear` clears the policy. This is an operator opt-in for classes of calls. It is not a teammate clicking approve on one sequence. Read it before setting it. Do not clear it to get a refused send through.
