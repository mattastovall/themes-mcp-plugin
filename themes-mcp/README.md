# Themes MCP plugin

One package for Codex and Claude Code. It installs the authenticated Themes MCP
server plus a shared workflow skill that keeps generation results grid-first and
prevents duplicate references, duplicate widgets, and oversized inline payloads.

The plugin connects to `https://mcp.themegen.ai` using the host's OAuth flow.
It contains no API keys and does not bundle the legacy bot server.

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
