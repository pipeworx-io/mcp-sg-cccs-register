# @pipeworx/sg-cccs-register

Singapore Competition and Consumer Commission (CCCS) public register of cases against businesses: infringement decisions and penalties for price fixing, bid rigging and information exchange, abuse of dominance investigations, consumer-protection actions on misleading practices, and commitments accepted from firms. Merger reviews are not served.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `sg_cccs_cases_search(query?, category?, limit?)` -- searches the register, newest update first. The keyword matches the case title and firm name. `category` is `agreements_collaborations`, `dominance_conduct`, `consumer_protection` or `commitments_remedies`. Each hit: case id, title, category, link. `individuals_dropped` counts the entries that name a person and were not returned. A search with no hit is `found:false` with `empty_reason: no_match`; a source that cannot be read is an error.
- `sg_cccs_case_get(id)` -- one case by its id (`category/slug` as returned by the search): published title, status (for example `Investigation | Closed - Infringement Decision Issued`), CCCS reference, decision date, the case summary naming the firms, penalties mentioned, and links to the decision and media release. An unknown id is `found:false`; an entry naming an individual is `found:false` with `empty_reason: not_a_firm_case`.

Every response carries `source` and `data_as_of`.

## Auth

Keyless.

## Data sources

- Competition and Consumer Commission of Singapore, public register. Read live per call, nothing stored.

Privacy: firms only. Register entries that name a person (an ex-director penalised, a named individual in a joint venture) are dropped and counted. A case summary lists the parties the CCCS names; sole proprietorships appear under their business name as the CCCS publishes it.

Traps worth knowing: the register mixes decisions with proposals and consultations, so check `status` in the case before treating an entry as a finding; search titles are rebuilt from the case address, the case get returns the published title; a zero-hit search means no match, not that the business has no CCCS history.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sg-cccs-register": {
      "url": "https://gateway.pipeworx.io/sg-cccs-register/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sg-cccs-register/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/sg_cccs_cases_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"bid rigging"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sg_cccs_cases_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-sg-cccs-register` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/sg-cccs-register/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "sg-cccs-register": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sg-cccs-register"]
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
ask_pipeworx({ question: "your question about Sg Cccs Register data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
