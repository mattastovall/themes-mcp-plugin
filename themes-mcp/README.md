# Themes MCP plugin

One package for Cursor, Codex, Claude Code, and OpenCode. It installs the authenticated Themes MCP
server plus a shared workflow skill that keeps generation results grid-first and
prevents duplicate references, duplicate widgets, and oversized inline payloads.

The plugin registers the `themes` server and connects to
`https://mcp.themegen.ai` using the host's OAuth flow. It contains no API keys
and does not bundle the legacy bot server.

For Cursor, install or copy this folder to
`~/.cursor/plugins/local/themes-mcp/`, then reload Cursor so it registers the
bundled MCP server. Older Cursor versions can also use the same `themes` entry
in `~/.cursor/mcp.json`.

## Manifest layout

| File | Consumer |
| --- | --- |
| `plugin.json` + `mcp.json` | Portable Agent Plugins package (OpenAI). OpenAI presentation metadata lives under `extensions.com.openai.interface`; `mcp.json` requires `type: "streamable-http"`. |
| `.codex-plugin/plugin.json` + `.mcp.json` | Codex compatibility fallback. Ignored by hosts that read the root `extensions.com.openai` block. |
| `.claude-plugin/plugin.json` + `.mcp.json` | Claude Code. |
| `.cursor-plugin/plugin.json` + `mcp.cursor.json` | Cursor. |
| `opencode.json` | OpenCode (copy the `mcp.themes` entry into your config). |

Keep `name`, `version`, and the endpoint identical across manifests;
`scripts/__tests__/themes-mcp-cursor-plugin.test.mjs` enforces this.

The bundled Cursor manifest explicitly points at `mcp.cursor.json`, not the
portable `mcp.json`, because Cursor's server entries use the portable URL form
(`url` only); do not add a `type` field to `mcp.cursor.json`.
For local debugging, copy the `themes-dev` entry from `mcp.dev.json` into
Cursor's MCP settings instead of replacing the hosted `themes` entry.

For local MCP debugging, run `bun run mcp:dev` and add the separate
`themes-dev` entry from `mcp.dev.json` to Cursor. It listens on
`http://127.0.0.1:3010/api/mcp`, while OAuth uses the canonical
`https://mcp.themegen.ai` audience so the hosted Themes consent page can issue
a token the local verifier accepts. Protected-resource metadata remains local
for Cursor compatibility; this audience is an explicit development alias, not
a production auth bypass. Production remains the `themes` server at
`https://mcp.themegen.ai`. Set `THEMES_MCP_DEV_OAUTH_RESOURCE` only when your
development environment has a different trusted HTTPS audience.

## Local development

Claude Code:

```sh
claude --plugin-dir ./plugins/themes-mcp
```

Or register the local marketplace:

```sh
claude plugin marketplace add ./plugins
claude plugin install themes-mcp@themes-web
```

Codex:

```sh
codex plugin marketplace add ./plugins
codex plugin add themes-mcp --marketplace themes-web
```

The Codex marketplace name is taken from `plugins/.agents/plugins/marketplace.json`.
The Claude plugin can be loaded directly with `--plugin-dir`; a marketplace entry
is included for repository hosting later.

Authentication happens when the host first connects to the MCP server. The
plugin intentionally does not configure a bearer header or copy image bytes
into tool arguments.

The shared skill also covers sequence-scoped editorial work: Fountain and
storyboard revisions, labeled shot batches, explicit candidate selection,
After Effects delivery approval, and asynchronous JSON/CSV/PDF exports.

## OpenCode

Add the `themes` entry from `opencode.json` to `opencode.json` in your project
or to `~/.config/opencode/opencode.json`, then authenticate:

```sh
opencode mcp auth themes
opencode mcp list
```

OpenCode discovers OAuth from the server's `401` challenge and protected-resource
metadata, then registers itself with Dynamic Client Registration (RFC 7591)
against the Supabase authorization server. Do not set `oauth` unless you
pre-register a client. If the flow fails, `opencode mcp debug themes` shows
which discovery step broke. As of 2026-09-30 production metadata advertises a
`registration_endpoint`; a full browser sign-in and consent has not been
verified from OpenCode itself.
