# Building the graph

Whole graph at once when the design is clear. Otherwise start empty and add one node at a time, reading the saved result after each. For a change to an existing workflow, edit only the node or setting the user named and leave the rest of the graph untouched. Remove a node by removing it, not by rebuilding the graph.

Before writing a node, read that node type's real settings and the workspace's real options for any choice field, such as a channel, a worksheet or an account. Reference earlier outputs and worksheet columns only in the form the settings describe.

Give every branch an explicit outcome, including the case where a value is missing. Avoid dead ends and orphan nodes.

Validate after every change, fix every real error and validate again. A workflow is ready only when validation is clean and the saved graph contains every requested node and path.

## Capability ids

| Step | Id |
| --- | --- |
| Find an existing automation | `workflow.automations.list` with `kind` `workflow` |
| Read the saved graph | `workflow.automations.get` with `kind` `workflow` and `automation_id` |
| Legal node kinds and subtypes | `workflow.automations.node_catalog` with `kind` `workflow` |
| Required config for one subtype | `workflow.automations.node_inspect` with `kind` `workflow`, `node_kind`, and `subtype`. Omit `automation_id`. An `automation_id` alone is refused: node_inspect describes a contract, not an instance. |
| Choice values for a field | `workflow.automations.field_options` |
| Tokens the draft may use | `workflow.automations.variables_list`, passing the draft graph when nothing is saved yet |
| Insert one node | `workflow.automations.node_add` with `kind` `workflow` |
| Change one node | `workflow.automations.node_patch` |
| Move or remove one node | `workflow.automations.node_move`, `workflow.automations.node_remove` |
| Replace the whole draft | `workflow.workflows.draft_save` |
| Check the draft | `workflow.automations.lint` with `kind` `workflow` |
| Freeze the draft | `workflow.workflows.draft_publish` |

`workflow.automations.node_add` with `kind` `sequence` is a sequence step. Do not use it to build a workflow. A workflow node is not flat: `id`, `type`, and `position` sit on the node, and `kind`, `subtype`, and `config` sit under `node.data`. The caller mints `node.id`. There is no delay node. A wait is `delay_seconds` on the action. `node_inspect` of subtype `delay` or `wait` returns not_found with the message `there is no delay node; set delay_seconds on the action node (task, message) to wait before it runs`.

`workflow.workflows.draft_save` replaces the entire draft. Nodes and edges you omit are gone. Read the current graph first and send it back with your edits. `expected_latest_draft_version_id` must match the stored draft, or the save raises `workflow_draft_conflict`. Pass null only when the workflow has never had a draft. The same idempotency key with a different graph fails `workflow_draft_idempotency_mismatch`. Lint can be clean and save can still reject the graph: exactly one trigger, no edge into the trigger, no duplicate ids, no cycles, every non-trigger node reachable, and at least one reachable action. A lone trigger with no edges is allowed on a draft and rejected on publish.

## Worked example

List workflows, then read the catalog:

```json
{ "kind": "workflow" }
```

That object is the params for `workflow.automations.list` and again for `workflow.automations.node_catalog`. Inspect the task subtype before inserting it:

```json
{ "kind": "workflow", "node_kind": "action", "subtype": "create_task" }
```

If `node_inspect` does not return `create_task`, stop and use a subtype the catalog returned. On a saved trigger-only workflow, add one task. Take `expected_latest_draft_version_id` from `workflow.automations.get`. Null is legal only when that read has no draft yet. Omit `after_node_id` only when the graph is the trigger alone. The server then returns `defaulted_after_node_id`.

```json
{
  "kind": "workflow",
  "automation_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "expected_latest_draft_version_id": null,
  "node": {
    "id": "task-1",
    "type": "action",
    "position": { "x": 0, "y": 160 },
    "data": { "kind": "action", "subtype": "create_task", "label": "Review the new row", "config": {} }
  }
}
```

Read back `added_node_id` (`task-1`). Then `workflow.automations.get` with `{ "kind": "workflow", "automation_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa" }` and confirm `task-1` is in the graph. Lint with `workflow.automations.lint` and the same `kind` and `automation_id`. Fix every error and lint again.

To write the whole draft in one call, `workflow.workflows.draft_save`:

```json
{
  "workflow_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "expected_latest_draft_version_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
  "graph": {
    "nodes": [
      { "id": "trigger-root", "type": "trigger", "position": { "x": 0, "y": 0 }, "data": { "kind": "trigger", "subtype": "table", "config": {} } },
      { "id": "task-1", "type": "action", "position": { "x": 0, "y": 160 }, "data": { "kind": "action", "subtype": "create_task", "config": {} } }
    ],
    "edges": [{ "id": "edge-trigger-task", "source": "trigger-root", "target": "task-1" }]
  }
}
```

Read back `definition.latest_draft_version_id`, `latest_draft_version.graph`, and `latest_draft_version.content_hash`. Publish only after that readback matches the requested nodes, using `workflow.workflows.draft_publish` with `expected_draft_version_id` from the save and `expected_active_version_id` null on the first publish. A mismatch raises `workflow_draft_version_mismatch`. Publish does not run the workflow.
