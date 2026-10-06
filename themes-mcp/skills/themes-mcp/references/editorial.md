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

