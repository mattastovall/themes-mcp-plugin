# Themes MCP plugin

One package for Cursor, Codex, and Claude Code. It installs the authenticated Themes MCP
server plus a shared workflow skill that keeps generation results grid-first and
prevents duplicate references, duplicate widgets, and oversized inline payloads.

The plugin registers the `themes` server from `mcp.json` and connects to
`https://mcp.themegen.ai` using the host's OAuth flow. It contains no API keys
and does not bundle the legacy bot server.

For Cursor, install or copy this folder to
`~/.cursor/plugins/local/themes-mcp/`, then reload Cursor so it registers the
bundled MCP server. Older Cursor versions can also use the same `themes` entry
in `~/.cursor/mcp.json`.

For local MCP debugging, run `bun run mcp:dev` and add the separate
`themes-dev` entry from `mcp.dev.json` to Cursor. It listens on
`http://127.0.0.1:3010/api/mcp`, while OAuth uses the canonical
`https://mcp.themegen.ai` audience so the hosted Themes consent page can issue
a token the local verifier accepts. Production remains the `themes` server at
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
