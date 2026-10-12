# Building the graph

Whole graph at once when the design is clear. Otherwise start empty and add one node at a time, reading the saved result after each. For a change to an existing workflow, edit only the node or setting the user named and leave the rest of the graph untouched. Remove a node by removing it, not by rebuilding the graph.

Before writing a node, read that node type's real settings and the workspace's real options for any choice field, such as a channel, a worksheet or an account. Reference earlier outputs and worksheet columns only in the form the settings describe.

Give every branch an explicit outcome, including the case where a value is missing. Avoid dead ends and orphan nodes.

Validate after every change, fix every real error and validate again. A workflow is ready only when validation is clean and the saved graph contains every requested node and path.
