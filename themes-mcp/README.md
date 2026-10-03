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

## Branding and schema support (0.1.4)

The portable manifest and Codex compatibility overlay both reference the bundled
Themes logo (`assets/themes-icon.png`) and composer icon (`assets/themes-icon.svg`).
These are copies of the application's existing `public/apple-touch-icon.png` and
`public/favicon.svg`; the brand color is `#FF7E1D`. The package keeps the same name,
MCP endpoint, OAuth connection, and default prompts.

The server supports MCP `2026-07-28` (`server/discover`, per-request metadata,
matching method/name headers, stateless results) alongside legacy `2025-11-25`
initialization. Tool input and supported output schemas use JSON Schema 2020-12.
Modern responses carry the server's title, website, and icons in serverInfo metadata.
Only implemented capabilities are advertised; optional subscriptions, sampling,
elicitation, and tasks are not advertised as supported.

After deploying the server update, its public OpenAPI 3.2.1 description is available
at `https://mcp.themegen.ai/mcp/openapi.json`. It is built directly from the MCP tool
contracts and describes the real JSON-RPC `POST /` endpoint, OAuth bearer challenge,
protocol headers, notifications, and errors. Tool names are documented in
`x-mcp-tools` with input schemas and the stable output schemas currently published
by `tools/list`; they are not separate REST routes. OpenAPI tools that only accept
3.0 schemas need a compatible importer; do not downgrade schemas by relabeling them.

Updating this directory or producing an archive does not deploy the server or
refresh an installed host's plugin cache. Use the existing release/install flow
for the target host, then reconnect to refresh its tool catalog.

## Pinned media workspace (0.1.5)

`themes_open_workspace` accepts `{}` and declares both global and thread OpenAI
MCP App entrypoints, with the title **Media workspace** and a monochrome sidebar
icon. Opening it lists only the connected account's owned threads, with search
and pagination. Selecting a thread opens its existing gallery without generating
media or spending credits; **All threads** returns to the workspace.

Both resource contents and MCP Apps initialization declare `inline` and
`fullscreen` display modes. **Expand workspace** requests fullscreen from the
host; **Compact view** returns to inline. The views use the mode actually granted
by the host and follow subsequent host-context notifications. Fullscreen uses a
scrollable viewport rather than growing the inline card. Grid v20 and sequence
shell v5 retain their previous resource URI aliases for existing conversations.

The sidebar/tab entrypoints require the updated MCP server to be deployed and
the host's tool catalog to refresh. Updating the plugin ZIP alone does not add
them to an already-connected server.

## Durable requests and inspection (0.2.0)

Thread restoration includes an attributed request ledger and private revisioned
view state. The Requests view searches retained server history; feed search only
filters loaded media. Default agent context is capped near 2,000 tokens, mixing
recent intent and relevant older requests. Automatic routing considers substantive
activity within 24 hours and creates a new thread for ambiguous matches. Explicit
accessible threads and exact generation/request/batch links remain usable at any age.

Video inspectors load metadata first. Explicit Analyze video requests prepare up
to eight timestamped frames through the media inspection worker. MCP agents inspect
returned evidence themselves; web analysis uses the configured server analyzer.
Sampling does not establish complete motion continuity or audio content. Draft
restoration never authorizes or submits a revision. Grid v23 retains earlier aliases.
