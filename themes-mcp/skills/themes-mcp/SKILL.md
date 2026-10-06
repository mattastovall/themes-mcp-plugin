---
name: themes-mcp
description: Create, revise, inspect, and organize images and videos in Themes. Use reviewed model prompting guidance, owned references, coherent batches, and the interactive generation grid.
---

# Themes creative workflow

Use the authenticated Themes MCP server for model discovery, references,
generation, and saved work. Start from the user's creative intent and preserve
their constraints through each generation and revision.

## 1. Establish the work

Respect an explicit thread or accessible Themes media link. Otherwise call
`themes_resolve_thread_request` with the original user request and a stable
source key. Carry the returned thread and `ledgerRequestId` into generation.
Load `themes_get_thread_context` when continuing existing work. Historical
requests provide context; they do not authorize new paid work.

Identify the desired output, required content, reference roles, and constraints.
Use reasonable defaults for minor creative choices. Ask only when missing
information materially changes the result. A request to generate authorizes
generation; opening a workspace, inspecting media, or restoring a draft does not.

## 2. Choose the model and read its guidance

Honor a named model. Otherwise use `themes_list_models` for the intended output
and input mode, preferring compatible favorites. `kind` means image or video;
`inputMode` describes text, image, or multimodal input topology.

Call `themes_inspect_model` for the exact model and input mode. Read the route,
reference requirements, controls, and `advancedSettingsBucket`. Final endpoint
selection belongs to the server and depends on the actual inputs.

Before writing model-specific prompts, call `themes_get_prompt_guidance` with
the model ID and route mode. Apply available reviewed structure, techniques,
pitfalls, examples, and dialect metadata. If guidance is unavailable, use the
inspected contract and ordinary visual description; do not invent provider
syntax. Catalog descriptions are discovery metadata, not approved guidance.

## 3. Compose the prompt and references

- Describe concrete subject, action, setting, composition, lighting, and style
  where relevant. For video, describe how action and camera movement unfold.
  Include sound only when requested and supported by the inspected route.
- Preserve exact requested text and explicit exclusions. Guidance helps express
  the request; it does not override the user's creative choices.
- Explain what each reference controls: identity, layout, appearance, or a
  starting/ending frame. Use supported reference fields rather than inventing
  image-index tokens or unsupported syntax.
- Keep settings in the inspected settings bucket. Put visual content in the
  prompt, ordering in `items[]`, and human labels in supported metadata fields.
- Keep shot/frame numbering and overall-count labels out of ordinary prompts.
  Contact-sheet generation, multi-cam mode, and automatic tile splitting are
  disabled for MCP. Use ordinary single-image generation or separate batch items.

Upload or ingest a reference once and reuse its owned `referenceId`.
Use `items[].referenceId` for per-slot inputs. Never send image bytes, base64,
data URLs, or large serialized responses as MCP arguments.

For saved Themes or LoRAs, search with `themes_search_themes`, then inspect the
selected UUID with `themes_inspect_theme` for the actual model and inputs.
Attach only when requested, using `themeSelection` with `themeId` and the
inspected `snapshot.hash`. Apply relevant trigger words and inspected scale
notes; never construct provider LoRA URLs. “Use Themes” names the product and
does not itself request a saved Theme attachment.

For H3 LoRAs, search with `loraOnly: true`, `loraFamily: minimax-h3`,
`output: video`, `loraModality: video`, and the intended `videoInputMode`.
Inspect video/audio reference compatibility. Reinspect if the snapshot drifts.

## 4. Generate and wait

Use one `themes_generate_batch` call for four or more distinct prompts.
`numImages` repeats one prompt; it does not represent distinct shots.
For a direct batch, show a table of item numbers and prompts before dispatch.
Wait for approval only when the user requested a review gate. Check each prompt
against their request, references, and chosen look before sending it.

Give each independent request a distinct idempotency key. Preserve the key and
unchanged payload for exact retries. Poll `themes_get_generation` using the
returned request ID; never submit another generation merely to poll. A queued
receipt is not ready media. Reconcile uncertain outcomes before another spend.
Retry failed bulk slots on the original key; completed slots must not be spent
again. Preserve batch identity and `bulkRowIndex` ordering.

Use server-issued `generationRef` values for generations and `referenceId`
values for owned references. Request IDs are for polling. Row IDs, provider
IDs, `unique_id`, and batch IDs do not substitute for generation identity.

## 5. Inspect and revise

Generation, character preparation, confirmation, revision, and polling return
metadata without opening an inline panel. After dispatch, call
`themes_render_generation_grid` once with the returned thread ID to show it.
For a batch, pass its returned `bulkRunId` to display that exact pass.
For one image, pass its returned `generationRef` to show only that generation,
not its thread history. This scope is preserved by refresh and polling.
Do not open the workspace or render intermediate character setup and polling.
The mounted grid refreshes its thread. Open another panel only when the user
explicitly asks to reopen or switch work.

For a storyboard batch, wait for HTTPS media in every slot, compose a labeled
vision grid with `themes_compose_vision_grid`, then use `themes_interpret_media`
before claiming visual completion. For existing stills, use that same inspection
path rather than sending the stills to a second vision model.

Revise failed cells with `themes_revise_batch_items` in the original batch.
State what should change and what should remain consistent. For character
continuity, establish a hero still and character sheet, then use the returned
`characterTokens`; arbitrary grid cells are not character identity.

For a chain of separate edits, use exactly the prior chosen successful result's
`media.imageUrl` as the next `referenceImagesUrl`. Do not substitute a thumbnail,
thread preview, failed result, or request ID. Use Show all generations when
comparing the newest unbatched chain with an older batch.

On explicit video analysis, prepare inspection with
`themes_prepare_media_inspection`, poll `themes_get_media_inspection`, and
inspect the returned timestamped frames. Samples cannot prove complete motion
continuity or audio content. Opening an inspector does not start analysis.

## 6. Return the result

Keep the response short: identify the thread/batch, say whether media is ready
or still processing, and point to the already-open grid. Name observed issues
and useful revisions when inspection supports them. Use
`themes_get_generation_details` for saved settings or measured metadata;
requested resolution is not measured resolution.

Normal results belong in the grid; do not repeat them as Markdown image embeds.
When the user explicitly requests inline images, call
`themes_get_generation_details` with `includeImage: true` and exactly one
`generationRef` per call, then surface the native image attachment. For several
requested images, use one read-only call per image. If attachments cannot be
shown, use the already-mounted grid. If neither surface is available, provide
exact `media.imageUrl` links. URL previews must use the exact original
`media.imageUrl`, never `media.thumbnailUrl`; thumbnails may not be ready.
Never reconstruct, truncate, or abbreviate media URLs. A missing attachment or
failed preview does not authorize regeneration.


## Specialized workflows

Read only the relevant supporting file before using these workflows:

- [After Effects](references/adobe.md): authenticated sessions, opaque targets,
  typed operations, bindings, workflow permissions, and terminal receipts.
- [Editorial sequences](references/editorial.md): scripts, immutable boards,
  prepared passes, candidate selection, reviewed delivery, and exports.
- [Thread context and drafts](references/thread-context.md): attributed request
  history, reconnects, revision conflicts, and private view state.
