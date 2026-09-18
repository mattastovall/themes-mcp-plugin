---
name: themes-mcp
description: Use when generating, revising, inspecting, or organizing image/video work through the Themes MCP server. Keeps MCP calls compact, avoids duplicate references and duplicate widgets, and makes grid results visible without unnecessary follow-up calls.
---

# Themes MCP workflow

Use the authenticated `themes` MCP server for Themes generation and media
inspection. Treat the server as the authority for model routes, generation
identity, references, batches, and grid state.

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
