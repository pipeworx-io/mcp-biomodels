# @pipeworx/biomodels

BioModels (EMBL-EBI) — the public repository of curated, executable
mathematical models of biological systems (SBML, CellML, BioPAX and more),
searchable by pathway, disease, organism, gene or author, with the model files
themselves and the paper each model came from.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `biomodels_search(query, limit?, offset?)` — find published models by pathway,
  process, disease, organism, gene or author, with facet counts by curation
  status, modelling approach and cross-referenced ontology terms.
- `biomodels_model(model_id)` — one entry in full: SBML notes, curation status,
  modelling approach (with its MAMO term), the source publication, contributors
  and version history.
- `biomodels_files(model_id)` — every attached file with MIME type, size,
  MD5/SHA-256 and a direct download URL: the primary SBML plus auto-generated
  BioPAX, Octave/MATLAB, VCML and SciLab conversions.

## Auth

Keyless. No registration step.

## Data sources

- <https://www.biomodels.org/search?query=&format=json> — faceted model search.
- <https://www.biomodels.org/{modelId}?format=json> — one model's metadata.
- <https://www.biomodels.org/model/files/{modelId}?format=json> — file listing.
- <https://www.biomodels.org/model/download/{modelId}?filename=> — file bytes.

### Not the same service as `biostudies`

BioStudies is EMBL-EBI's general repository for the data supporting a
publication (any assay, any format); BioModels is specifically executable
models. Neither is a search surface for the other, and BioModels ids (`BIOMD…`,
`MODEL…`) are not BioStudies accessions (`S-…`).

### Things that cost time to rediscover (measured 2026-09-17)

- **The host in every old doc is a redirect.** `https://www.ebi.ac.uk/biomodels/…`
  answers 301 to `biomodels.org`, which redirects again to `www.biomodels.org`.
  This pack calls the final host directly rather than depending on two hops
  being followed with the query string intact.
- **`format=json` is required** — without it you get HTML with a 200.
- **`numResults` has a floor of 10 and does not clamp downward.** Asking for 2
  or 3 returns 10 rows; asking for 25 returns 25. A caller that trusted the
  parameter would report 10 results as "the top 3". This pack truncates the list
  itself and always requests at least the floor so `offset` paging stays aligned.
- **`offset` is row-based and does work** — page with `offset += limit`.
- **`facetStats` is a JSON string containing JSON**, not an object.
- **`modellingApproach` is an object** (`{accession, name, resource}` over a
  MAMO term), not a bare string; `publication.accession` is the PubMed ID when
  `publication.type` says "PubMed ID".
- **A model's `description` is an SBML `<notes>` XHTML blob**, not prose.
- **Search latency is spiky** — the same query measured 0.6s, 5.5s and >25s
  within one minute, so the default 25s fetch bound reports an up service as
  down. The search path uses 45s; by-id endpoints are sub-second.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "biomodels": {
      "url": "https://gateway.pipeworx.io/biomodels/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/biomodels/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/biomodels_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"glycolysis","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/biomodels_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "biomodels": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-biomodels"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-biomodels
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Biomodels data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
