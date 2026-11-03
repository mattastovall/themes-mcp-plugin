---
name: story-production
description: Plan, produce, review and resume character-driven story ads, animated explainers and narrative films in Themes. Use for scripts, coverage, cast and set continuity, pilot stills, approved motion, shot review, and production handoff.
---

# Story production in Themes

Use this workflow when the user asks for a narrative production or opts into staged production. Ordinary image requests use themes-mcp. Read that skill for generation, identity, polling and recovery contracts.

## Establish the production

Continue the explicit accessible thread and sequence. A thread can contain multiple sequences; never guess the current sequence. Record the brief, exact supplied copy, audience, placement, aspect, tone, existing assets, exclusions and authorized budget. Ask only about missing decisions that materially change the work. Preserve supplied copy; flag timing conflicts instead of shortening it silently.

Use the production review tools only when advertised by the connected server. An installed skill does not prove the backend is deployed. If unavailable, save the brief and coverage plan as thread Markdown, identify the missing capability, and do not claim an enforced approval gate or publish an invented board URL.

When available, use these boundaries:

- `themes_save_story_production`: enroll the exact sequence/board revision and save the production metadata, export naming, external handoff, verification evidence and reported cost state. Use `expectedRevision: null` only for the first record; use the returned revision for subsequent saves. Approval fields are deliberately excluded.
- `themes_get_story_production`: resume the current persisted record and inspect selected still identities and its handoff manifest. Always carry the explicit `sequenceId` and `boardRevisionId`.
- `themes_open_story_production`: show the compact review and its exact revision link when the user asks to review it. Data reads and saves do not open another widget.
- The authenticated board records user approval and prepares motion without dispatch. Carry its returned pass ID into `themes_dispatch_sequence_pass` only when generation is authorized. Dispatch rechecks the approved selections. Never imitate approval by writing shot metadata.

The sequence is the durable production identity. Keep one review board URL for it, update its revision, retain earlier rounds, and identify the exact version being reviewed. A board is private under the thread's access rules; sharing is a separate user instruction.

## Develop before generating

1. State what the viewer should feel, understand and do. Analyze references as timestamped picture/sound/strategic beats; adapt their structure without copying distinctive characters or lines.
2. Write the premise, cast roles and look locks, tone, recurring motifs, and reusable locations. Treat product appearance and claims as facts from inspected user sources. Mark unresolved claims for review.
3. Establish a line table with stable line IDs, speaker, exact caption text, optional distinct spoken text, selected take and measured timing. The recorded read sets duration.
4. Map each set into stations and named cameras. A shot names line IDs, set, station, framing, angle, action, background, continuity group and reference roles. Read [coverage](references/coverage.md).
5. Test a small representative hook and shared body before scaling. After a tone or scale change, make a master and three or four pilot stills; review them before generating the full set.

## Stills and the review checkpoint

Establish owned character and set references, then compose one first frame per shot with room for motion and safe text placement. Use inspected route support for cast/set/product inputs. Do not flatten reference roles into a single arbitrary reference or silently omit inputs to fit a model.

Prepare immutable passes without spend, inspect warnings and pricing, then dispatch within the user's authorized scope. Persist request IDs before polling, preserve exact retry keys, and reconcile uncertain provider outcomes. Do not infer zero spend from an error or missing provider ID. Never regenerate to repair a missing preview.

Update the existing review board after each completed or rejected round, before reporting the result. Every shot carries its line, timing, location, framing, selected candidate and concrete review note. Show all current frames, prior rounds, master references and relevant continuity pairs. If an update fails, state that the board is stale and retain the results for recovery.

“Agent-ready” is an assessment. User approval is a separate attributed decision about exact source revisions and selected stills. Selection alone is not approval. Obtain the user's still approval before motion for this staged workflow. Replaced stills or changed relevant source invalidate the affected approval; do not carry it forward implicitly. Preserve approvals for unchanged work only when the server verifies their scope.

## Motion and edit

Prepare motion from the approved still for each shot. Use the current model contract for first-frame or reference-video inputs. Describe subject motion, camera motion and what stays fixed. Split unreliable complex action into shots with consistent screen direction. Inspect start/middle/end frames; report that sampled frames do not establish complete playback or audio quality.

Voice, music, mixing and native editing may require an external tool. Discover available capabilities and explain the specific gap before using an alternative. Record tool, source assets, settings, output identities and actual verification. Never claim external processing merely because it appears in the plan.

Use one auditioned voice per character, version line takes, transcribe tricky product names, and preserve delivery dynamics. Time picture and captions to measured narration. Render at the source cadence unless a deliberate conversion is validated. Draw approved labels, captions, disclaimers and UI in the edit; keep client end cards clear of added overlays.

## Review, revision and handoff

Read [review and handoff](references/review-and-handoff.md). Separate narration sync from visual craft. Verify a criticism against the source before proposing a fix. Prefer a crop, retime or existing asset when it addresses the actual problem. Paid fixes require scoped authorization and a current estimate; the existence of a rejection is not permission to spend.

Touch only the requested shots and lines. Save a new revision and retain source lineage. End each production session with a current board and resumable handoff: exact sequence/revision/pass IDs, selected media, external editable source, dependencies, decisions, costs including unknown outcomes, verification evidence and next steps.
