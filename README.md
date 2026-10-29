# Wisconsin Statutes by citation

Keyless. `940.01` is first-degree intentional homicide.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Why this one is assembled, not sliced

Wisconsin heads its History and Note blocks with the same section number and
places them BEFORE the statute, so any positional anchor lands on editorial
apparatus. The markup instead tags each subsection with the citation it belongs
to (`data-cites="statutes/940.01(1)(a)"`), so the text is assembled from those
spans. When a page tells you what each fragment is, use that rather than
inferring from position.

## Stored copy and `data_as_of` (fleet #3581)

The Legislature's site answers US data centres but refuses or stalls from
Singapore: over 7 days only 7 of 176 external calls succeeded (US callers
57 of 57). A cache cannot help a first touch, so the statutes are stored, one
JSON object per chapter, in the shared statute store (R2, `statutes/wi/`) that
the Illinois, Indiana, North Carolina and Wyoming packs also use. The gateway
hands the pack its R2 binding and the pack reads only the chapter a call names.
A section found there is answered with no upstream request; the response says
`served_from: "stored-copy"` and carries `data_as_of` (when that chapter was
last read from the source). A section the copy cannot answer (chapter missing,
or listed with no stored text) falls back to the live read with its short-lived
cache. (fleet #3596: the copy used to be 470 files in git, 47.7 MiB in every
worktree, re-uploaded to KV on every gateway deploy.)

Nothing is loaded at module scope: the data never enters the gateway bundle.

## Refresh

```
node mcps/wisconsin-code/scripts/bake-chapters.mjs            # all chapters -> mcps/wisconsin-code/.bake/
node mcps/wisconsin-code/scripts/bake-chapters.mjs --only 940 # one chapter
node mcps/wisconsin-code/scripts/bake-chapters.mjs --resume   # skip chapters already on disk
node mcps/wisconsin-code/scripts/upload-chapters.mjs          # publish to R2 (statutes/wi/), changed chapters only
```

Sequential, with a delay between requests and an honest user agent. Exit 0
only if every chapter in the table of contents was fetched and parsed and the
totals clear the floors (default 400 chapters, 12,000 sections with text); any
section that cannot be fetched leaves its chapter's previous file untouched
and the run exits 1 with the counts. A chapter whose text is unchanged keeps
its previous `data_as_of`. The baked files are a working copy, not committed
(`.bake/` is gitignored); `upload-chapters.mjs` writes only the chapters whose
content hash differs from the one recorded in `statutes/wi/_manifest.json`, so
a refresh that finds nothing new writes nothing. It uses
`wrangler r2 object put --remote` under the operator login; no API token.

## Data source

Wisconsin Legislature, Wisconsin Statutes. Wisconsin statutes are public
record; this pack reads the published statutes and republishes no
annotations.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "wisconsin-code": {
      "url": "https://gateway.pipeworx.io/wisconsin-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/wisconsin-code/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/wi_statute \
  -H 'Content-Type: application/json' \
  -d '{"section":"940.01"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/wi_statute`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-wisconsin-code` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/wisconsin-code/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "wisconsin-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-wisconsin-code"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Wisconsin Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
