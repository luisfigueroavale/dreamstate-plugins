# Manifest shape (not executable here)

The catalog has no apply, list, or remove capability for this shape. Writing it down does not register an API. Do not pass it to `integrations.integrations.upsert` or to `tables.columns.add` and call that registration.

If a future capability appears in `tools_search`, `tools_get` its schema and follow that schema. Until then, refuse the register, list, and remove requests in one sentence.

A complete manifest, for the user's own notes, has:

| Field | Rule |
| --- | --- |
| `slug` | Lowercase letter, then letters, digits, underscores. At most 63 characters. Same rule as attach. |
| `name` | Display name. |
| `baseUrl` | Public `https` origin. Private hosts are not a supported target. |
| `operations` | At least one. |

Each operation has `operation`, `method` (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`), and `path` starting with `/`. `effect` defaults to read for GET and write otherwise. A non-GET cannot be declared read.

Auth blocks you may describe, and must not collect as chat secrets:

| Auth | What the lease would hold |
| --- | --- |
| `none` | No attach |
| `bearer` | Token for an Authorization bearer header |
| `api_key_header` | Named header and optional prefix |
| `api_key_query` | Named query parameter |
| `basic` | Username in the manifest, password only in the lease |

Field mapping, when an operation runner exists later: `key`, `type` (`string`, `number`, `boolean`, `string[]`, `number[]`, `object`, `object[]`, `array`), `required`, `location` (`path`, `query`, or `body`), and optional `wireKey`. Path fields replace `{name}`. Query fields become URL parameters. Body fields become JSON. Location default is query for GET and body otherwise. A GET has no body.

`tables.columns.add` does not accept this manifest. Input mapping from a worksheet column to an operation is not available until a real tool id shows up in discovery. Never invent a custom_http tool id from the slug.
