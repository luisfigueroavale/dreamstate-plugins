---
name: workflows
description: "Best practices for building and running automations that are not outreach sequences: frame the job, smallest graph, validate after every change, runtime inputs, safe recovery of failed runs."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Workflows

## Boundary

- Owns: automations that are not outreach sequences.
- Multi-step outreach sequences: [`outreach`].
- Explaining failed or stuck runs: [`diagnostics`].
- The `workflow` pack lists the tools and their exact inputs.

## Choose the path

| Job | Path |
| --- | --- |
| A cadence that messages people, even one message | A sequence. Use [`outreach`]. `workflow.automations.node_add` with `kind` `sequence` is a sequence step and is out of scope here. |
| The same trigger, branches, and actions every time | A workflow. Build it with the ids in [graph-authoring.md](graph-authoring.md). |
| An open-ended goal that picks the next action from what it finds | Not a workflow graph. The catalog exposes `agents.run_get`, `agents.runs_list`, and `agents.run_cancel` for `command_agent.turn` runs only. Any other run kind is not found. There is no create-an-agent capability. Do not imitate an agent by saving a workflow, and do not claim an agent run was started. |

Reply wording and sending in an existing thread belong to [`inbox-replies`]. [live-replies.md](live-replies.md) only says how a workflow hears that a reply happened. Retry, continue, and stop are in [durable-runs.md](durable-runs.md).

## Practice

Best practices for building and running automations. The workflow pack lists the tools and their exact inputs; this skill is how to use them well. Outreach cadences to people are sequences, covered by the outreach skill. Everything else that is deterministic is a workflow.

1. Build new unless asked. A new request gets a new automation built from scratch. When the user names an existing one to change, resolve that name among existing automations by listing workflows and sequences with that name as the search text, then change the match. A named thing that is not a workflow or sequence may be a task, a record or a worksheet; check those only after automations come back empty. A step inside a workflow, such as a task step, is edited in that workflow's graph, not in the task list.
1a. "Enroll these in a workflow" is never a missing input. Use a workflow whose table trigger is the worksheet the user is looking at, or one they named; otherwise build a new draft for that worksheet (worksheet rows trigger plus fitting steps, a follow-up task per person by default), validate, and enroll. Never ask which workflow. Filters on any worksheet column narrow who enrolls; with no filter every row enrolls. When the user said who should enroll, filter on the columns that hold it and report how many enrolled and how many were held back.

Social account post and AI answer worksheets hold content rows, never an audience or subject. To reach authors, send unique author URLs to a people worksheet first.
2. Frame the job before building: record the trigger, input schema, step aliases, exact capability ids, expected outputs, failure behavior, idempotency, cost, stopping condition and frequency. Distinguish deterministic workflows from adaptive agent goals.
2a. Use a blueprint when one does exactly what you need; otherwise build your own. Workflow blueprints already cover common jobs: CRM syncs, list and do-not-contact syncs, routers, digests and signal monitors. List blueprints of the workflow kind, read what the closest one creates and what it still needs, install it, then adapt it to this request. Installing never needs answers or connections; the result names what is still to pick, so say that. Build from scratch only when no blueprint fits, and say which one came closest.
3. Ask one clarifying question only when a required input is missing. Otherwise build with sensible defaults and say what you assumed.
4. Build the smallest graph that delivers the result. Build the whole graph in one pass when you know every node. Add one node at a time when exploring, and read each result before the next.
5. Read a node's real settings before writing them and use real workspace options. Never invent variable syntax, node ids or field names. Conditions point at real worksheet columns.
6. A value that changes on every run becomes a runtime input, not a hardcoded setting. When a value should come from the trigger, the triggering record or an earlier step, list the variables for the graph you are about to save, passing the draft graph itself when nothing is saved yet, and use a listed token. Never save an empty or null value for something the user asked for; say which variable is missing instead.
7. Validate after every change. Fix every real error and validate again. Never call a workflow ready until validation is clean and the saved graph shows every requested node and path.
8. Building never waits on connections. A missing Slack, email or other connection blocks activation only. Build, edit and validate anyway, then name the connection to add.
8a. Building never waits on rows. A worksheet trigger on an empty or still-filling worksheet is still a valid enrollment: build, validate and turn it on, and rows enroll as they land. Say it is waiting for rows, never stop because the worksheet is empty.
8b. Use the row-created trigger when each new worksheet row must enter once. It watches future row arrivals on one worksheet, carries that row identity through the graph, and does not reuse or freeze a saved selection. Use the table trigger only when user wants a selected saved-view audience.
8c. To tag or mark a triggering row, add the worksheet-row update step. Choose its column and either replace its value or append a value to its existing list. Do not ask user to export, reimport, or manually tag rows when workflow already has triggering row identity.
8d. Stripe and PostHog events start workflows through a product event trigger on that provider's signal source. Stripe events arrive named with a stripe prefix and the Stripe event type, for example invoice payment failed, subscription trial will end, subscription deleted and invoice paid; read the exact names from the source's captured events. PostHog events arrive with the event name captured in PostHog. Read the signal source list for the source id and use the exact event name; never invent one. Each event enrolls the person once, and a duplicate delivery of the same event never enrolls twice. To act on the person's worksheet row, the row is already updated in place by its Stripe or PostHog source; add a worksheet-row update step only for a value the event itself does not set.
8e. Event weights decide how much each product or billing event moves a lead's signal score. Read and change them on one signal source with the event weights tools. Every event starts at weight 0, which records the event but does not move the score, and older events count less over time. Set a weight only for events the user named as buying signals, and read the weights back after writing them.
9. Activating something that sends or spends follows the paid-run consent rule.
10. Draft, published, active, running and finished are separate facts. Report only what the last result proves.

## Validate and run safely

Use `workspace.blueprints.list` with `{ "kind": "workflow" }`, then `workspace.blueprints.describe` with the returned `blueprint_id`. Where the selected installable system supports it, use `workspace.blueprints.preflight` with `blueprint_id`, a stable `request_key`, and `params`; inspect missing prerequisites and the disabled install plan. Use `workspace.blueprints.apply` with `blueprint_id` and the described `inputs`. Read the resulting draft and unresolved inputs. Preflight availability depends on the selected blueprint, so inspect its schema and result rather than assuming every blueprint supports it.

Run `workflow.automations.lint` on the candidate document while drafting and on the saved `automation_id` before publication. This interface does not expose draft and publish phase flags: saved lint resolves workspace blockers and tokens, while document lint cannot resolve variables. Fix errors, distinguish unknown readiness from a missing prerequisite, and read warnings. Lint is advisory and does not replace save or publish validation.

Use `workflow.automations.preview` with `kind: "workflow"` and `automation_id` in live read mode. It runs nothing. Inspect external writes, connections, credit floor and floor confidence; a floor is neither a quote nor a ceiling. Resolve unknown node contracts. For an active immutable version, `workflow.workflows.rehearse` records a zero-effect trace up to the first action; its optional `node` marks a target only if the prefix can reach it. It is not a full single-node execution. Where supported, smoke-test `workflow.workflows.call` against the saved revision, then dry-run the same input and revision. Inspect projected effects and required parameters; neither starts a run. Set an explicit credit ceiling, follow the paid-run consent rule and obtain any required approval, then perform a small live pilot only when requested. Read the run and its trace before scaling. If any mode or target is unsupported, report the limitation instead of claiming a node was tested.

Use `workflow.workflows.trigger_events_catalog` to select an available event and its actual fields and readiness. `workflow.triggers.create` accepts a campaign and existing graph node; `workflow.workflows.trigger_create` provisions credentials for an existing workflow trigger node and normally returns `owner_approval_required` outside its owner-approved root path. Neither creates the graph node. Table triggers use enrollment, not provisioned trigger rows. Validate schedule syntax and the current service limits before activation; do not copy limits from another product. Legacy workflow webhook triggers are retired. Do not promise workflow webhook secret rotation or replay without a matching capability for that exact resource; inspect the appropriate resource and its current schema instead.

### Example: review a manual workflow call

For a saved workflow returned as `automation_id`, call `workflow.automations.lint` with `{ "kind": "workflow", "automation_id": "<id>" }`, then `workflow.automations.preview` with the same params in live read mode. Resolve any lint error and inspect the returned revision and credit floor. For its active workflow id, call `workflow.workflows.rehearse` in live mode with `{ "workflow_id": "<id>", "input_json": {} }` and an idempotency key; read the returned run. Call `workflow.workflows.call` with `{ "workflow_id": "<id>", "input_json": {} }` first in smoke-test, then dry-run mode. If the output shows the intended bounded effect and the user requested a live pilot, satisfy its returned required parameters and the paid-run consent rule before the live call. Read `workflow.workflows.runs_list` with `{ "workflow_id": "<id>" }` to resolve the workflow subject, then read `workflow.workflows.run_trace_get` with `{ "workflow_id": "<id>", "subject_id": "<returned subject id>" }`. Report actual effects, cost and any unresolved outcome. Never retry an uncertain effect until its receipt is reconciled.

## Delete and archive

When user says delete, delete permanently. Show impact first unless user already asked to delete. Archive only when user says archive. Never say hard deletion is unavailable.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.
