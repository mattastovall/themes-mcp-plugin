## After Effects connector

The Adobe client is an authenticated Themes account installation. Route every
After Effects request through Themes MCP; never connect an agent to CEP, a local
socket, or the host directly. Start with `themes_adobe_list_sessions`, then
choose an explicit `sessionId` and `documentId` from the same account. The
active selection, document name, layer index, and layer name are display hints,
not write targets.

- Read context with `themes_adobe_get_context`. Use the bounded summary first,
  then request the project tree, composition layers, layer properties, source
  metadata, or markers with the returned opaque IDs and pagination cursor.
- Capture a frame, selection, or source with `themes_adobe_capture_reference`,
  then pass the returned owned reference to the normal Themes generation tools.
- Import owned generations, references, batches, or batch slots with
  `themes_adobe_import`. Supply an explicit composition placement when needed;
  never pass an untrusted media URL as the source authority.
- Apply only typed edits with `themes_adobe_apply_operations` (rename,
  visibility, transform, text, timing, and placement). Do not request arbitrary
  ExtendScript, expressions, effect graphs, destructive deletion, or document
  reconstruction.
- Inspect bindings with `themes_adobe_list_bindings`; use
  `themes_adobe_update_bindings` to pin, resume, or unlink. Batch bindings use
  `(bulkRunId, bulkRowIndex)` and update only after the complete relevant
  revision set is ready.
- Poll every mutation with `themes_adobe_get_operation` until `completed`,
  `failed`, `cancelled`, `outcome_unknown`, or `recovery_required`. Preserve the
  same idempotency key for retries and never replay an operation whose host or
  remote outcome is uncertain.

The workflow tools expose the existing AE helpers through typed operations:
`themes_adobe_transcribe`, `themes_adobe_beat_this`,
`themes_adobe_read_mapping`, `themes_adobe_matanyone2`,
`themes_adobe_reframe`, `themes_adobe_upscale`,
`themes_adobe_remove_silence`, and `themes_adobe_create_master`. Use
`inspect`/`preview` actions for read-only planning; `run`, `load`, or `generate`
actions require the appropriate write permission, and remote processing also
requires `generate`. Every workflow needs explicit AE target IDs and returns an
operation ID before host application.

Permission boundaries are account-scoped: `adobe_read` permits session,
context, binding, operation, inspect, and preview reads; `adobe_write` permits
captures, imports, edits, binding changes, and host mutations;
`generate` is additionally required for transcription, Beat This, Read Mapping,
MatAnyone2 loading, generative reframe, upscale, and Create Master. A missing
permission is an authorization failure, not evidence that the AE session is
offline.

