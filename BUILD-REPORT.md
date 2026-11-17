# BUILD REPORT — caseprep

Rebuild of `com.legalheadquarters/legal-hq` v1.0.1 (remote-only) as
`@reneturcios/caseprep` 0.1.0 / `io.github.CloudCorpRecords/caseprep`.

## Fixture conflicts

**None.** The spec and the fixture
(`harness/fixtures/legalheadquarters-tools-original.json`) agree on all
4 tool names, descriptions, and schemas:

- `about_legal_hq` — NO required key
- `get_starter_kit` — NO required key
- `get_case_prep_checklist` — NO required key
- `get_setup_session` — NO required key

All four tools take no arguments: every `inputSchema` is exactly
`{type: "object", properties: {}, additionalProperties: false}`, with no
`required` key and no `title` key. The fixture's tool literals (name,
description, inputSchema only — `annotations` dropped at slimming, per
the fixture header) are served verbatim from `src/forward.ts`. The
`TOOL_DEFS` literals were generated programmatically from the fixture
file (parse + re-serialise via
`/tmp/laneb4-caseprep-scratch/gen_tools.py`), never retyped by hand.

## What was built

All 19 factory files under `builds/caseprep/`:

- `package.json` (`@reneturcios/caseprep` 0.1.0, `mcpName`
  `io.github.CloudCorpRecords/caseprep`, bin `caseprep` → `dist/cli.js`,
  scripts build=`tsc` / test=`tsx --test test/*.test.ts` / dev; deps
  exactly `@modelcontextprotocol/sdk ^1` + `zod ^3`; devDeps
  `@types/node ^20`, `tsx ^4`, `typescript ^5`)
- `server.json` (description exactly the prescribed 91-char string, see
  below; npm entry carries `"version": "0.1.0"`; OCI entry
  `ghcr.io/cloudcorprecords/caseprep:0.1.0` carries **no** version field)
- `Dockerfile` (node:20-alpine both stages; `ENTRYPOINT ["node",
  "/app/dist/cli.js"]`, `CMD ["mcp"]`; server-name label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/caseprep"`
  matches `server.json`)
- `tsconfig.json` (fleet-identical, 318 bytes), `.gitignore`,
  `package-lock.json`, `README.md` (fresh prose, no copied text),
  `LICENSE` (MIT, Copyright (c) 2026 Rene Turcios), `CHANGELOG.md`
  (single 0.1.0 entry, 2026-10-28)
- `src/index.ts`, `src/cli.ts`, `src/mcp.ts`, `src/types.ts`,
  `src/forward.ts` (core; exports `VERSION = "0.1.0"`)
- `test/forward.test.ts`
- `.github/workflows/{ci.yml, publish.yml, publish-container.yml,
  registry-publish.yml}` (container workflow: IMAGE
  `ghcr.io/cloudcorprecords/caseprep`, smoke assertion on server name
  `caseprep`)

## Design

- Forwarding client to `https://legalheadquarters.com/mcp` (pinned),
  overridable via `LEGALHEADQUARTERS_MCP_URL` (**labelled
  reconstruction**). tools/call is forwarded as a streamable-HTTP
  JSON-RPC client: initialize (protocolVersion `2025-03-26`, clientInfo
  `caseprep`) → notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; session established lazily and reused
  per client. Plain-JSON and SSE replies both parsed; the matching-id
  message wins; upstream results pass through verbatim (including
  `isError` and extra fields such as `structuredContent`).
- **No Authorization header is ever sent**, under any environment: the
  upstream registry record declares no headers. The forwarding core only
  sets `Content-Type`, `Accept`, and (when issued) `Mcp-Session-Id`.
  Tests assert the absence of any Authorization header under several
  environments: empty env, env-override set, credential-like env values
  present (`LEGALHEADQUARTERS_API_KEY`, `AUTHORIZATION`, `API_KEY` —
  all ignored), explicit URL override, and across a reused session.
- Upstream failures (JSON-RPC error, non-2xx status, malformed body,
  throwing/timing-out fetch) surface as `caseprep error:` text results
  (**labelled reconstruction**).
- Registration via the MCP SDK **low-level** `Server` API
  (`ListToolsRequestSchema` / `CallToolRequestSchema` handlers) serving
  the fixture literals exactly; no `McpServer` + zod registration.
  `src/mcp.ts` also exports `createServer` / `startMcpServer` /
  `handleCallTool` so the MCP layer is testable without a transport;
  the CLI's `mcp` subcommand starts it over stdio.
- Core takes an injectable fetch (`ForwardDeps.fetchFn`), so all tests
  run against a stubbed JSON-RPC upstream. Fake-fetch tests only; no
  live network calls in tests; no live call claimed anywhere.

## Labelled reconstructions

- `LEGALHEADQUARTERS_MCP_URL` env override for the pinned upstream
  endpoint.
- `caseprep error:` failure rendering.
- CLI shape (one subcommand per tool, `--version`/`-v`, exit codes
  0/1/2) following the fleet factory pattern.

## Lockfile (Route 3, clean from birth)

Generated in the build dir with no `node_modules` present:
`npm install --package-lock-only --ignore-scripts --offline`.
Verified: parses, `name == @reneturcios/caseprep`, **exactly 128
package entries**, sha256
`a22bb5802c4e126cfa6f5ef8f2e46f57a98a901152edd5f972c667e7c3668bfc`.
Then `npm ci --ignore-scripts --offline` populated `node_modules`
(exit 0).

## server.json description

- Text: "Legal HQ public reads: about, the $99 Starter Kit, the case-prep checklist, setup sessions."
- Character count: **91** (measured programmatically).

## Verification (all in the build dir)

- `npx tsc --noEmit` → exit 0
- `npm run build` → exit 0; real `dist/` (cli.js 2343 B, forward.js
  10045 B, mcp.js 2665 B, all non-empty, with `.d.ts` + source maps)
- `node dist/cli.js --version` → `0.1.0`
- `npm test` → **41 tests, 41 pass, 0 fail** (8 suites: registration
  integrity — served tools/list deep-equals the fixture's 4 tools in
  order, required fields per tool, empty-properties schemas; session
  establishment/reuse/session-id handling; no-Authorization-header
  assertions under several environments; env URL override; verbatim
  argument forwarding for all 4 tools plus unfiltered caller args;
  verbatim result pass-through incl. upstream `isError` and SSE;
  JSON-RPC/init/malformed/HTTP/throw/timeout failures →
  `caseprep error:`; parseRpcBody; MCP-layer validation; CLI
  `--version`/`-v`/unknown exit 2/bad-JSON exit 2/array exit 2)
- Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py
  --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/legalheadquarters-tools-original.json
  --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/caseprep/dist/cli.js mcp"
  --out /home/hatch/workspace/mcp-rebuild-100/builds/caseprep/parity-out`
  → **PASS, parity 100.0, 4/4 identical, no missing/extra/schema
  diffs** (see `parity-out/parity.json`)
- Consistency sweep: package.json version = server.json top-level =
  npm-entry version = OCI tag = core `VERSION` = CHANGELOG heading
  (0.1.0); package.json `mcpName` = server.json `name` = Dockerfile
  label; server.json description measured at 91 chars; OCI entry
  confirmed to have no `version` field.

Clean-room: all prose and code written fresh; only the fixture's tool
literals are served verbatim.
