---
description: Reopen the Themes grid for an existing thread without creating a duplicate widget.
---

Reopen the Themes grid for the current or explicitly named existing thread.

- If a generation, batch, revision, character sheet, or grid-generation tool
  ran in this turn and returned widget metadata, surface that existing result
  instead of calling another render tool.
- Otherwise call `themes_render_generation_grid` with the known thread ID.
- Do not generate, poll, or fetch raw history just to make the grid visible.
- If no thread ID is available, ask for the thread title or use the normal
  thread-search tool first.
