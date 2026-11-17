# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Legal HQ MCP server, serving all 4 captured tools locally for tools/list and forwarding every tools/call upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when issued).
- Tools: `about_legal_hq`, `get_starter_kit`, `get_case_prep_checklist`, `get_setup_session`. All four take no arguments and return published information.
- Upstream results pass through verbatim; upstream failures surface as `caseprep error:` text results. No Authorization header is ever sent: the upstream record declares no headers.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). Upstream endpoint overridable via `LEGALHEADQUARTERS_MCP_URL`.
