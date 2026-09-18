# Themes plugin marketplace

This directory is the source of truth for the public [`themes-mcp-plugin`](https://github.com/mattastovall/themes-mcp-plugin) repository.

The public repository mirrors this directory into its marketplace root:

- `themes-mcp/` is the installable plugin.
- `.agents/plugins/marketplace.json` is the Codex marketplace manifest.
- `.claude-plugin/marketplace.json` is the Claude marketplace manifest.

For local development from this repository:

```sh
claude plugin marketplace add ./plugins
claude plugin install themes-mcp@themes-web

codex plugin marketplace add ./plugins
codex plugin add themes-mcp --marketplace themes-web
```

The plugin connects to the hosted Themes MCP server at `https://mcp.themegen.ai`.
