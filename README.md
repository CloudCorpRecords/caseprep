# caseprep

caseprep is a small local bridge to Legal HQ's hosted MCP
server. Legal HQ publishes plain information for business
owners who are facing a lawsuit or a legal dispute: what
the service is, a starter kit for getting a case organised,
a checklist to work through before hiring a litigation
lawyer, and a paid one-hour session to set everything up.

The bridge stores nothing locally. It answers the tool
list itself, using the tool definitions exactly as
captured from the live endpoint, and passes every tool
call upstream, handing the upstream's answer back
untouched. All four tools are plain published-information
reads: none of them takes an argument, submits a form,
takes payment details, or sends personal information
anywhere. The Starter Kit tool returns a price and a
purchase link as data; following that link is the
reader's own choice, outside this tool.

## Tools

- `about_legal_hq` — what Legal HQ is and who it helps:
  AI legal help for business lawsuits, ways to keep lawyer
  costs down, getting organised for a lawyer, and finding
  a lawyer on contingency. No arguments.
- `get_starter_kit` — the fine print on the $99 Legal HQ
  Starter Kit: what is included, what it costs, and where
  it can be bought. No arguments.
- `get_case_prep_checklist` — a short, free list of what a
  business owner should pull together before meeting or
  hiring a litigation lawyer. No arguments.
- `get_setup_session` — the details of Legal HQ's paid
  one-hour setup session, where Claude is connected to the
  user's email and files and a case record is built
  together. The tool only returns the published offer; it
  books nothing. No arguments.

## Configuration

- `LEGALHEADQUARTERS_MCP_URL` (optional) — override the
  upstream endpoint. Defaults to
  `https://legalheadquarters.com/mcp`.
  (The override hook is this rebuild's own labelled
  reconstruction.)

No API key is used and no `Authorization` header is ever
sent: the upstream server's registry record declares no
headers, and this bridge adds none of its own.

### Forwarding mechanics

On the first tool call the client opens a streamable
HTTP session with the upstream: a JSON-RPC `initialize`
request (protocolVersion `2025-03-26`, clientInfo name
`caseprep`), then a `notifications/initialized`
notification, then the `tools/call` request carrying the
caller's arguments unchanged. When the upstream answers
`initialize` with an `Mcp-Session-Id` header, that value
rides on every later request of the session, which is
reused for the life of the process. Replies may arrive
as plain JSON or as server-sent events; both are read.

Successful upstream results pass through verbatim,
including any `isError` flag the upstream sets.
Failures — a JSON-RPC error, a non-2xx status, an
unreadable response, or a request that cannot be
completed — come back as an `isError` result whose
text starts with `caseprep error:` (this rendering
is this rebuild's own labelled reconstruction).

## CLI

Each tool is also a subcommand. All four take no
arguments, so the JSON argument string can simply be
omitted:

```bash
npx @reneturcios/caseprep about_legal_hq
npx @reneturcios/caseprep get_starter_kit
npx @reneturcios/caseprep get_case_prep_checklist
npx @reneturcios/caseprep get_setup_session
npx @reneturcios/caseprep --version
```

Exit codes: `0` on success, `1` on an `isError`
result, `2` on a usage error.

## MCP server

```bash
npx -y @reneturcios/caseprep mcp
```

MCP client config:

```json
{
  "mcpServers": {
    "caseprep": {
      "command": "npx",
      "args": ["-y", "@reneturcios/caseprep", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/caseprep
docker run --rm ghcr.io/cloudcorprecords/caseprep --version
```

The container's default command is `mcp`, so the first
form is a stdio MCP server; any other arguments reach
the CLI normally.

## Development

```bash
npm install
npm run build
npm test
npm run dev
```

The test suite runs entirely against a stubbed
JSON-RPC upstream — no live network calls.

## License

MIT — see `LICENSE`.
