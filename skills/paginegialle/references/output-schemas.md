# Output schemas

Every data command emits exactly one JSON document on stdout. This file
documents the shape of each. Fields marked *optional* may be absent on a
given record.

## `search`

```json
{
  "schema_version": 1,
  "query": { "what": "idraulici", "where": "roma", "page": 1 },
  "items": [
    {
      "id": "h24multiserviceidraulicih24aroma",
      "name": "H24 Multiservice - Idraulici H24 a Roma",
      "categories": ["idraulici e lattonieri"],
      "address": {
        "street": "Via Arimondi Giuseppe, 21",
        "postal_code": "00159",
        "city": "Roma",
        "province": "RM"
      },
      "phones": ["393 423261391", "340 1054226"],
      "snippet": "Idraulico Roma H24 per perdite, guasti, ...",
      "rating": { "value": 3.8, "reviews": 17 },
      "url": "https://www.paginegialle.it/h24multiserviceidraulicih24aroma"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "has_more": true,
    "next_page": 2
  }
}
```

- `schema_version` — integer, currently `1`.
- `query` — echoes the effective query (`what`, `where`, `page`).
- `items[]` — one object per business:
  - `id` — stable listing id.
  - `name` — display name.
  - `categories` — list of category labels.
  - `address` — `street`, `postal_code`, `city`, `province` (province code,
    e.g. `RM`). Some fields may be empty strings.
  - `phones` — list of phone numbers (may be empty).
  - `snippet` — short description *optional*.
  - `rating` — `{ "value": <float>, "reviews": <int> }` *optional*.
  - `url` — canonical profile URL; a slug for `get` can be taken from here.
- `pagination` — `page`, `per_page`, `has_more`, `next_page`.

## `get`

A bare JSON object (no `items` wrapper):

```json
{
  "id": "ristorante-al-mago-albairate-mi",
  "name": "Ristorante al Mago",
  "categories": ["Ristoranti"],
  "legal_name": "Ristorante al Mago",
  "vat": "10539550151",
  "tax_id": "10539550151",
  "address": {
    "street": "Frazione Castelletto, 27",
    "postal_code": "20080",
    "city": "Albairate",
    "region": "Lombardia"
  },
  "geo": { "lat": 45.40046, "lon": 8.93629 },
  "phones": ["+39 02 94960166", "+39 320 6832426"],
  "email": "info@ristorantealmago.com",
  "website": "https://www.almagoristorante.com",
  "social": ["https://www.almagoristorante.com"],
  "rating": { "value": 5, "reviews": 2 },
  "opening_hours": [
    "Su 12:30 - 14:30, 20:00 - 23:30",
    "We 12:30 - 14:30, 20:00 - 23:30"
  ],
  "description": "A lor Signori, Amici e Parenti ...",
  "url": "https://www.paginegialle.it/ristorante-al-mago-albairate-mi"
}
```

Fields are drawn from the profile page's schema.org JSON-LD. Common optional
fields: `legal_name`, `vat`, `tax_id`, `geo`, `email`, `website`, `social`,
`rating`, `opening_hours`, `description`. Missing means the source didn't
publish it.

## `contacts`

```json
{
  "schema_version": 1,
  "query": { "what": "idraulici", "where": "roma", "page": 1, "max": 50 },
  "items": [
    {
      "name": "Professione Clima",
      "categories": ["Impianti idraulici e termoidraulici"],
      "address": {
        "street": "Via di Casalotti, 251/B",
        "postal_code": "00166",
        "city": "Roma",
        "province": "RM"
      },
      "phones": ["06 61568396"],
      "email": "info@example.it",
      "website": "https://www.example.it",
      "vat": "01234567890",
      "url": "https://www.paginegialle.it/professione-clima-frasca-co"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "has_more": true,
    "next_page": 3
  },
  "stats": {
    "requested": 50,
    "enriched": 50,
    "with_email": 31,
    "with_website": 44
  }
}
```

- `query` additionally echoes `max`.
- `items[]` — same core fields as `search`, plus `email`, `website`, `vat`.
  These three are present only when enrichment found them, or when
  `--enrich=false` is passed (in which case they are omitted entirely).
- `stats` — `requested` (asked for), `enriched` (profiles fetched),
  `with_email`, `with_website`. Use these to report coverage — not every
  business publishes an email or website.

## Errors

On failure, stdout is empty and a JSON error object is written to **stderr**.
Exit code is `1` for runtime failures and `2` for usage errors (missing
`--what`/`--where`, bad subcommand, etc.). Always check the exit code before
trusting stdout.
