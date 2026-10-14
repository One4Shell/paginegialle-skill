---
name: paginegialle
description: >-
  Search Italian businesses and build enriched B2B contact lists from
  paginegialle.it using the paginegialle CLI bundled at scripts/paginegialle.
  Use this whenever the user wants to find Italian companies or professionals
  by category and city, look up a specific business's address, phone, email,
  VAT number or website, or produce a lead / contact list for B2B outreach in
  Italy. Trigger even when the user does not name the tool, e.g. "trova
  idraulici a Roma", "find restaurants in Milan", "get me contacts for law
  firms in Turin", "search for elettricisti near Naples", or "look up this
  paginegialle profile". Only for paginegialle.it / Italian business-directory
  lookups — not for general web search or non-Italian directories.
license: MIT
compatibility: linux-amd64
---

# paginegialle

Wraps the `paginegialle` CLI, a client for the Italian business directory
[paginegialle.it](https://www.paginegialle.it). It searches listings and
enriches company profiles into stable, machine-readable records for B2B
contact research.

## Prerequisites

- The executable **is** bundled at `scripts/paginegialle` (relative to this
  skill's directory). Run it via that path, e.g.
  `./scripts/paginegialle search --what idraulici --where roma`.
- It is a statically linked **linux/amd64** binary. On another OS/arch it will
  not run — in that case tell the user instead of trying workarounds.
- It reads paginegialle.it over the network. No API key is required.

## Output contract

- Data commands emit **exactly one compact JSON document on stdout** and
  nothing else (JSON is the default; `--human` switches to tables).
- Errors are **structured JSON on stderr**.
- Exit codes: `0` success, `1` runtime failure, `2` usage error.

Parse stdout as JSON (e.g. pipe to `jq`). Never scrape the human tables when
you need values — they are for display only.

## Core commands

| Command | Purpose | Required flags |
|---------|---------|----------------|
| `search` | List businesses by keyword + location | `--what`, `--where` |
| `get` | Fetch and enrich one full business profile | `<slug\|url>` positional |
| `contacts` | Search + normalize into B2B contact records | `--what`, `--where` |
| `version` | Print CLI version | — |

### `search` — listings

```bash
./scripts/paginegialle search --what idraulici --where roma
./scripts/paginegialle search --what "centro estetico" --where milano --max-pages 3
./scripts/paginegialle search --what ristoranti --where torino --fields name,phones,url
```

- `--what` category/keyword, `--where` city/area (both required).
- `--page` start page (default 1); `--max-pages` how many pages to fetch
  (default 1).
- `--fields` comma-separated subset of item keys to keep (shrinks output).
- Response has `items[]` and `pagination.has_more` / `pagination.next_page`.

### `get` — one enriched profile

```bash
./scripts/paginegialle get ristorante-al-mago-albairate-mi
./scripts/paginegialle get https://www.paginegialle.it/osteria-macondo-roma
```

- Argument is a profile **slug** (from a previous `search` item's `url`) or a
  full profile URL.
- Returns a bare JSON object (not wrapped in `items`): legal name, VAT/tax
  id, address, geo, phones, email, website, social links, rating,
  opening hours and description.

### `contacts` — B2B list

```bash
./scripts/paginegialle contacts --what idraulici --where roma --max 50
./scripts/paginegialle contacts --what "studi legali" --where milano --max 20 --enrich=false
```

- `--max` total contacts to return (default 25), `--page` start page.
- Enrichment (`--enrich`, default `true`) fetches each profile to add email,
  website and VAT. Pass `--enrich=false` for a fast, search-only run (no
  email/website/VAT in the records).
- `--concurrency` parallel fetches when enriching (default 1; raise carefully).
- Output includes `stats` (`requested`, `enriched`, `with_email`,
  `with_website`) so you can report coverage.

## Global flags (all commands)

| Flag | Meaning | Default |
|------|---------|---------|
| `--json` | machine-readable JSON | on |
| `--human` | human-readable tables | off |
| `--delay` | minimum delay between requests, e.g. `1s` | CLI default |
| `--cache-ttl` | on-disk response cache TTL, e.g. `24h` | CLI default |
| `--no-cache` | disable the on-disk cache | off |
| `--retries` | max attempts per request on 429/5xx | CLI default |
| `--timeout` | per-request timeout, e.g. `30s` | CLI default |
| `--contact` | URL/email embedded in the User-Agent | — |
| `--base-url` | override site base URL | paginegialle.it |

Env vars mirror several flags: `PAGINEGIALLE_BASE_URL`,
`PAGINEGIALLE_CONTACT`, `PAGINEGIALLE_RETRIES`, `PAGINEGIALLE_TIMEOUT`.

## Workflow

1. **Identify the two axes**: `--what` (category/keyword — translate the
   user's intent to an Italian term if needed, e.g. "plumbers" → `idraulici`,
   "law firms" → `studi legali`) and `--where` (Italian city/area).
2. **Choose the command**:
   - Broad listing / exploration → `search`.
   - One known business → `get <slug|url>`.
   - A usable outreach list with emails/websites → `contacts`.
3. **Run the command** via `./scripts/paginegialle …`; keep JSON on stdout.
4. **Parse with jq** rather than eyeballing. See `references/examples.md` for
   ready-made recipes (extract names+phones, filter by email presence, count
   coverage).
5. **Throttle and cache**: for multi-page or batch runs, pass `--delay 1s`
   (or more) and let the response cache do its job — do not disable it with
   `--no-cache` unless the user needs fresh data. Set `--contact` to the
   user's URL/email so requests carry an honest User-Agent.
6. **Report** results as a table or list, and state the coverage stats when
   using `contacts` (not every record has an email/website).

## Recipes

**B2B lead list (with contact details):**

```bash
./scripts/paginegialle contacts --what idraulici --where roma --max 50 --delay 1s \
  | jq -r '.stats, (.items[] | [.name, (.phones|join(", ")), (.email//"-"), (.website//"-")] | @tsv)'
```

**Search then enrich the top hit:**

```bash
slug=$(./scripts/paginegialle search --what "centro estetico" --where milano --fields url \
  | jq -r '.items[0].url')
./scripts/paginegialle get "$slug"
```

**Paginate a whole result set:**

```bash
./scripts/paginegialle search --what ristoranti --where torino --max-pages 5 \
  | jq -r '.items[] | .name'
```

## Reference files

- `references/output-schemas.md` — exact JSON shape emitted by each command
  (field-by-field). Read it before writing parsers that depend on a field.
- `references/examples.md` — real captured outputs and `jq` recipes.

## Troubleshooting

- **Usage error (exit 2)** — a required flag is missing (`--what`, `--where`)
  or the subcommand is misspelled. Re-check the invocation.
- **Runtime failure (exit 1)** — read the JSON error on stderr; common causes
  are a bad slug for `get` or a network/HTTP failure. Retries and timeouts are
  configurable via flags.
- **`get` returns less than expected** — enrichment depends on what the
  profile page publishes; missing email/website/VAT means the source doesn't
  expose it, not a tool error.
