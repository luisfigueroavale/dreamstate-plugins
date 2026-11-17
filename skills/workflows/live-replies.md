# Live replies

Reply content, classification, and sending in an existing thread belong to [`inbox-replies`].

A workflow that should react after a reply uses a `sequence_event` trigger. Feed one outcome with `workflow.workflows.sequence_event_ingest`. Params are `sequence_id`, `contact_id`, `event_type` (`reply`, `completion`, `failure`, or `unsubscribe`), and `source_event_key`. It records the outcome and enqueues listening workflows. It never sends the reply and never pauses, resumes, or unenrolls the sequence. `workflowsEnqueued` 0 means no live graph was listening. `replayed` true means that `source_event_key` was already ingested.
