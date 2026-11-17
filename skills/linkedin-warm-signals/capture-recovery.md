# When a capture stalls

Read the status for the source you armed. Do not arm a second source of the same kind on the same worksheet.

| Read | What it tells you |
| --- | --- |
| `visibility.linkedin.ingestion_status` | Saved cursor, latest run status, landed rows, last error, and provider reads remaining today. It does not call LinkedIn. A spent daily budget waits until the next day. |
| `visibility.linkedin.network_capture_status` | Network capture pace. A long first walk is the drip, not a failure. |
| `visibility.linkedin.engagement_status` and `visibility.linkedin.engagement_harvest_status` | Health, rows written, and the last error for an engaged-with source. They do not call LinkedIn. |
| `visibility.linkedin.viewers_status` and `visibility.linkedin.followers_status` | Progress for that collection. |
| `visibility.linkedin.intent_status` | Saved engagement kinds and whether the row ceiling is armed. `configured` false means nothing is saved, so the worker still keeps both reactions and comments and applies no ceiling. |
| `tables.profile_watcher.get` | `watcher` null means this worksheet has no watcher. |

Fix the returned error, then read the same status again.

- `dependency_not_ready` with `source.linkedin_connections network capture is paused`, and the same sentence for `source.linkedin_followers` or `source.linkedin_new_connection`: the capture is paused. Resume that capture. Do not create another one.
- `workspace_account_unavailable` with `source.linkedin_connections requires an active workspace LinkedIn account` (same shape for followers and new connections): name the account that is missing. Building the worksheet can continue. Running cannot.
- `provider_session_unavailable` with `source.linkedin_connections account has no connected Unipile session`: the account has no session. Wait for a connected session. Do not switch the read onto a different account to get around it.
- `invalid_source_configuration` with `source.linkedin_followers requires a target_url`: set `target_url` to a real company or profile URL and run again.
- `invalid_source_configuration` with `source.linkedin_followers target_url is not a recognizable LinkedIn company or profile URL`, or `source.linkedin_followers could not resolve target_url to a LinkedIn provider id`: replace the URL. Do not retry the same string.
- `source.linkedin_jobs saved cursor is malformed; the original purchase needs recovery`, `source.linkedin_jobs cursor is malformed; the original purchase needs recovery`, or `source.linkedin_jobs cursor position is malformed`: stop the rerun. The purchase needs recovery. Do not buy the search again to hide the bad cursor.
- `crm.signals.search_run` readback `admission` `blocked`: fix every entry in `blocked_by`. Do not flip `approved` to true over a blocked dry run.
- `crm.signals.search_plan` with no route, including market-wide funding: tell the user that family and scope have no route. Offer `watch_list` only when the plan says that scope has a route.
- `tables.followers.order` budget exhaustion lands no partial rows. Read `tables.followers.status` with the same `request_id`. `follower_count` and `recorded_fees` are null until settlement. Null is not zero.

`visibility.linkedin.intent_setup` requires the worksheet to already have exactly one of `source.engaged_with_account`, `source.engaged_with_company`, or `source.engaged_with_team`. If that source is missing, attach it first. Do not point intent setup at a viewers import.

Report rows landed, the last error, and what remains. An empty page is not proof that nobody engaged.
