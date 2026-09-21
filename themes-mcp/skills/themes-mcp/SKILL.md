---
name: themes-mcp
description: Use when generating, revising, inspecting, or organizing image/video work through the Themes MCP server. Keeps MCP calls compact, avoids duplicate references and duplicate widgets, and makes grid results visible without unnecessary follow-up calls.
---

# Themes MCP workflow

Use the authenticated `themes` MCP server for Themes generation and media
inspection. Treat the server as the authority for model routes, generation
identity, references, batches, and grid state.

## Current server contract

- The server returns schema-versioned, compact generation snapshots. Do not
  expect settings, aliases, generation lineage, or inline media bytes in a
  normal generation result; use the dedicated inspection/detail tool when the
  user asks for saved settings or prepared metadata.
- Use server-issued `generationRef` values for generation identity. Keep a
  request ID only for polling `themes_get_generation`; never substitute row
  IDs, provider IDs, `unique_id`, or batch IDs.
- Batch summaries are paginated. Use `bulkRunId` for the complete batch and
  preserve `bulkRowIndex` for ordering; do not encode shot/frame numbering in
  prompts. Four or more distinct prompts belong in one
  `themes_generate_batch` call.
- Normal tool results are metadata-first and media URLs are HTTPS. Only
  vision/grid tools should attach pixels for inspection. Do not put image
  bytes, base64, or large serialized results in MCP JSON.
- Adobe tools operate through the authenticated account gateway, never a
  direct host socket. Writes require the session/document/opaque target
  context and must be polled until the Adobe receipt is terminal; `queued` or
  `running` is not proof that an edit applied.

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

## Editorial sequences

- Use an explicit `sequenceId` for every editorial call. Threads are the
  collaboration and access boundary; a thread may contain multiple sequences.
  Never infer a latest/current sequence.
- Use `themes_list_sequences` to enumerate a thread and
  `themes_get_sequence` to inspect one sequence; neither tool selects an
  implicit current sequence.
- The durable flow is `themes_create_sequence` →
  `themes_create_script_revision` → `themes_propose_board` or
  `themes_save_board_revision` → `themes_prepare_sequence_pass` →
  `themes_dispatch_sequence_pass` → `themes_list_pass_candidates` →
  `themes_select_pass_candidate`.
- A board shot always has a non-empty human label and a stable `shotKey`.
  Labels may repeat; labels are snapshots for passes, candidates, deliveries,
  and Adobe bindings. Use `shotKey`, never `label`, to join stages.
- Board revisions and approved delivery manifests are immutable. Send the
  expected source/content digest on mutating calls and reuse the exact
  idempotency key only for an exact retry. A new board or pass is a new
  revision, not an implicit latest update.
- Board proposals and reviewed board saves may assert the source script
  digest; use `origin: script_inferred` for the explicit script-to-board
  shortcut. Candidate selection and delivery approval also require their own
  idempotency keys; an approval replay must carry the same manifest digest.
- Shot-level video requires a materialized board revision. Continuous video is
  a separate one-asset pass and must not be represented as independently
  generated shots. Script-only continuous video is allowed; script-to-shot
  video must first materialize an inferred board for review.
- `themes_prepare_sequence_pass` is no-spend and preserves `sequenceId`,
  `passId`, source revision IDs, `shotKey`, label snapshots, `preparedBatchId`,
  and `bulkRunId`. Dispatch may spend balance and is owner-authorized; review
  all prepared prompts/warnings before dispatch.
- Delivery is staged: prepare, owner approve the exact `manifestDigest`, then
  execute through the authenticated AE gateway. The default is `new_master`;
  `existing_comp` requires an explicit AE comp target. Poll
  `themes_get_delivery` until every import, master-comp, placement, and
  read-back receipt is terminal. `automatic`, `pinned`, and `unlinked` binding
  policies remain meaningful; pinned/unlinked bindings must not be silently
  replaced.
- External HTTPS board references must become owned references before later
  generation or Adobe work. Durable identity is the owned reference or
  `generationRef`, never a URL. No audio generation/mixing is implied by the
  editorial metadata in v1.
- Use `themes_export_sequence` for the asynchronous sequence bundle and poll
  it with `themes_get_sequence_export`. Exports must preserve both `shotKey`
  and every historical label snapshot, including script, board, pass,
  candidate, delivery, provenance, and media digests. A successful export
  contains `sequence.json`, `shot-list.csv`, `shooting-script.txt`, and a
  readable `shooting-script.pdf`; use only the returned short-lived signed
  URL and never reconstruct storage keys.

## Result and grid behavior

- A generation tool that returns widget metadata already opens the grid. Surface
  that result to the user; do not call `themes_render_generation_grid` again in
  the same turn.
- Use `themes_render_generation_grid` only to reopen an existing thread when no
  generation, revision, character-sheet, or grid-generation tool ran in the
  current turn.
- Use `themes_generate_as_grid` only when the user wants one contact-sheet image
  split into cells. It is not the tool for browsing a thread.
- For a storyboard batch, wait for HTTPS media in every slot, then call
  `themes_compose_vision_grid` with the returned `bulkRunId`, followed by
  `themes_interpret_media`. Do this before declaring the batch visually done.
- For existing stills or references, compose a labeled vision grid and interpret
  it. Do not send the same stills to a second vision model.

## Compact payload rules

- Never put host image bytes, base64, data URLs, or large serialized tool
  responses in MCP JSON arguments.
- For owned stills, use the upload/commit flow and pass `referenceId` values.
  Use `items[].referenceId` for per-slot image-to-image inputs.
- Reuse one canonical owned reference per image. Do not repeat equivalent signed
  URLs, mirrored URLs, or lineage metadata in every generation.
- For four or more distinct prompts, use one `themes_generate_batch` call. Do
  not loop `themes_generate`.
- Preserve idempotency keys for exact retries. A new independent pass gets a
  new key; never replay a paid generation with a fresh key by accident.
- Prefer the structured result and its media links. Only pixel-inspection tools
  need image content attached by the server; normal history and generation
  results should stay link- and metadata-based.

## Identity and references

- Use the server-issued `generationRef` for generations.
- Use `ref:<id>` / `referenceId` for owned stills.
- Do not substitute row IDs, UUID request IDs, provider IDs, `unique_id`, or
  batch IDs for either identity type.
- Search and inspect a saved Theme only when the user explicitly names or asks
  to apply it. “Use Themes” means use the product, not attach a saved Theme.

## Editing and character continuity

- For source boards: ingest or upload once, attach references once, compose and
  interpret before dispatching edits.
- For character consistency: resolve a hero still and character sheet first,
  then use the returned `characterTokens`. Do not treat arbitrary grid cells as
  character identity.
- Revise failed cells with `themes_revise_batch_items` in the original batch;
  do not create a new fragment batch.

## User-facing completion

When returning a generation result, state the thread/batch identity, whether
media is ready or still polling, and point the user at the already-open grid.
For a thread reopened with the command, explicitly say that the grid was
reopened. Keep the response short; the grid carries the visual detail.
