---
name: collaboration
description: "Get a documented go or no-go from another person: comment threads on workspace objects, approval requests and approval rules, and who can decide what. It does not replace a paid run's own approval."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Collaboration

## Boundary
- Owns: Internal object discussions, approval rules, and review handoffs.
- The `unibox` pack lists the tools and their exact inputs.

Use this for an internal comment on a workspace object, a publishing approval rule, a content approval request or decision, a workflow approval that is waiting, a standing approval policy, or a workspace role change. Public comments on a published post belong to [`social`]. Customer replies belong to [`inbox-replies`]. Launching or enrolling a sequence belongs to [`outreach`]. A paid action's own preview and the paid-run consent rule are not replaced by a comment or by a rule.

Two mechanisms stay separate. A comment thread is a conversation about an object. Resolving it approves nothing. An approval is a decision a named person owes. It blocks a live path only where a rule, a content approval, a workflow approval, or the paid-run consent rule actually refuses the live call.

## Route

| Ask | Call | Not this |
| --- | --- | --- |
| Discuss a sequence, worksheet, row, knowledge page, or CRM record | `unibox.comment_threads.list`, then `unibox.comment_threads.create` | An approval rule |
| Is anything waiting on a person? | `workspace.observability.approvals_list` | Guessing from a resolved thread |
| Route future post submissions to publishers | `workspace.approval_rules_list`, then `workspace.approval_rule_create` | A comment |
| Ask named people to review this artifact | `social.content.approval.submit` | `social.content.approval.authorize_publish` when a rule named other approvers |
| Record the approver's decision | `social.content.approval.decide` | Deciding as someone else |
| A workflow run is paused for approval | `workflow.approvals.list`, then `workflow.approvals.decide` | A content approval |
| Who can do this? | `workspace.membership_get`, `workspace.members_list` | A job title |
| Opt-in extra approval on capability classes | `workspace.standing_approval_policy_get` | Inventing a sequence sign-off object |

The catalog has no capability that arms a sequence launch gate or a worksheet sign-off gate. `workspace.approval_rule_create` matches posts (platform, tag, author, and the other fields the schema allows) and routes them to publishing approvers. It does not block `sequence.sequences.launch`. A live sequence launch stays with [`outreach`] under the paid-run consent rule. If `workspace.observability.approvals_list` shows a pending item, use the command that item names.

Detail: [approvals.md](approvals.md) and [roles.md](roles.md).

## Safety ladder

Comments, rule lists, member lists, the review queue, and `workspace.observability.approvals_list` are reads or internal writes. They send nothing to a customer. `social.content.approval.submit` emails the named approvers. Confirm they are members and that the revision is current before submitting.

`workflow.approvals.decide` defaults `preview` to true. Read that preview, then decide with the same `plan_hash`. A changed preview needs a new review.

Role changes, rule create, and rule archive are workspace writes. Read the current role or rule first. Follow the paid-run consent rule when the schema or the refusal says the call is approval-bound. A viewer cannot perform growth mutations. Relay the refusal. Do not retry as if it were a transient error.

## Comments

`unibox.comment_threads.list` takes `object_type` (`sequence`, `table`, `table_row`, `knowledge_page`, `crm_record`), `object_id`, and `include_resolved` (default true). For a worksheet, `object_id` is `table_id`. For a row, `object_id` is `row_id`.

`unibox.comment_threads.create` requires `body` (1 to 10,000 characters). `title` is optional. A thread cannot be empty.

`unibox.comment_thread_comments.add` takes `thread_id`, `body`, and optional `parent_comment_id`. Resolve with `unibox.comment_threads.resolve`. Reopen with `unibox.comment_threads.reopen`. `unibox.comment_threads.delete` and `unibox.comment_thread_comments.delete` are permanent. Read the thread first and do not delete to "clean up" a decision you cannot find.

## Recovery

| What happened | What to do |
| --- | --- |
| `stale_revision` (409) on submit, decide, or authorize | Read the artifact revision again. Submit a new request for that revision. Do not reuse the old decision. |
| 403 on `social.content.approval.decide` | The caller is not the designated approver. Stop. Do not decide for them. |
| 409 `approval_already_decided` | A conflicting terminal decision already exists. Read the queue. Do not send the other decision. |
| `changes_requested` | The gate is not open. Edit, then submit a fresh review. The server opens a new round when the prior decision was invalidated by an edit. |
| Rule create returns `invalid_request` | An approver is not a member with publishing role `publisher` or `manager`. Read `workspace.members_list` and `workspace.membership_get`. |
| Observability source is null | That source could not be read. Say so. Do not treat null as an empty queue. |
| Workflow decide preview does not match the live intent | Stop. Decide only the `plan_hash` you just reviewed. |
| One-member workspace and a second person must approve | Invite them with `workspace.invitation_create` (role `admin`, `member`, `client`, or `viewer`). An invitation is not yet a role. Do not report `pending` as held. |

## Worked example

Read the caller's role with `workspace.membership_get` and `{}`. Read members with `workspace.members_list` and `{"limit":50}`.

List rules with `workspace.approval_rules_list`. If none route this post, create one only when the user asked for a standing rule:

```json
{"name":"LinkedIn posts need a publisher","conditions":{"match":[{"field":"platform","op":"eq","value":"linkedin"}],"operator":"AND"},"approver_user_ids":["cccccccc-cccc-4ccc-8ccc-cccccccccccc"],"require_all":true,"enabled":true,"priority":100}
```

The approver id must be a member whose publishing role is `publisher` or `manager`. Read the list again and confirm the rule id.

Submit the artifact review, not the rule, with `social.content.approval.submit`:

```json
{"artifact_id":"dddddddd-dddd-4ddd-8ddd-dddddddddddd","expected_revision":3,"approvers":[{"user_id":"cccccccc-cccc-4ccc-8ccc-cccccccccccc","routed_by":"rule","rule_id":"eeeeeeee-eeee-4eee-8eee-eeeeeeeeeeee","stage":"client_review"}],"needed_by":"2026-11-20T17:00:00.000Z","note":"Check the proof line and the offer."}
```

Read `approvals` and the bound revision. The approver reads `social.content.approval.review_queue` with `{}` and decides with `social.content.approval.decide`:

```json
{"approval_ids":["ffffffff-ffff-4fff-8fff-ffffffffffff"],"expected_revision":3,"decision":"changes_requested","reason":"The proof line names the wrong customer."}
```

`changes_requested` does not authorize publish. After the edit, submit again for the new revision. `decision` `approved` is the value that records approval. The word approve is the wrong field value. `social.content.approval.authorize_publish` writes a self-approved row with the caller as requester and approver and is what the publish gate reads. Do not call it to skip approvers an active rule named.

A comment on the sequence is a separate call and is not this approval. See the boundary example pattern: list threads, create a thread, list again. The new thread is not a launch approval.

## Decisions

- Read `opens` nothing from a resolved thread. Read the rule, the queue, and the live refusal.
- The designated approver decides. A 403 ends the attempt.
- An approval covers the revision it was bound to. An edit invalidates it.
- Expiry, when a request has `needed_by`, is a deadline on the answer. It does not expire an approval already recorded. If the content changed, ask again.
- Client role can read, comment, and decide what is assigned. It does not build the workspace. See [roles.md](roles.md).
