# Examples and recipes

Real captured outputs plus copy-paste `jq` recipes. All commands run the
bundled binary as `./scripts/paginegialle` from the skill directory.

## search — trimmed real output

```bash
./scripts/paginegialle search --what idraulici --where roma --fields name,phones,url
```

```json
{
  "schema_version": 1,
  "query": { "what": "idraulici", "where": "roma", "page": 1 },
  "items": [
    {
      "name": "H24 Multiservice - Idraulici H24 a Roma",
      "phones": ["393 423261391", "340 1054226"],
      "url": "https://www.paginegialle.it/h24multiserviceidraulicih24aroma"
    },
    {
      "name": "Professione Clima",
      "phones": ["06 61568396"],
      "url": "https://www.paginegialle.it/professione-clima-frasca-co"
    }
  ],
  "pagination": { "page": 1, "per_page": 25, "has_more": true, "next_page": 2 }
}
```

## get — enriched profile (real, abridged)

```bash
./scripts/paginegialle get ristorante-al-mago-albairate-mi
```

```json
{
  "id": "ristorante-al-mago-albairate-mi",
  "name": "Ristorante al Mago",
  "categories": ["Ristoranti"],
  "legal_name": "Ristorante al Mago",
  "vat": "10539550151",
  "tax_id": "10539550151",
  "address": { "street": "Frazione Castelletto, 27", "postal_code": "20080", "city": "Albairate", "region": "Lombardia" },
  "geo": { "lat": 45.40046, "lon": 8.93629 },
  "phones": ["+39 02 94960166", "+39 320 6832426"],
  "email": "info@ristorantealmago.com",
  "website": "https://www.almagoristorante.com",
  "rating": { "value": 5, "reviews": 2 },
  "opening_hours": ["Su 12:30 - 14:30, 20:00 - 23:30", "We 12:30 - 14:30, 20:00 - 23:30"],
  "url": "https://www.paginegialle.it/ristorante-al-mago-albairate-mi"
}
```

## contacts — B2B records (real, abridged)

```bash
./scripts/paginegialle contacts --what idraulici --where roma --max 3 --enrich=false
```

```json
{
  "schema_version": 1,
  "query": { "what": "idraulici", "where": "roma", "page": 1, "max": 3 },
  "items": [
    {
      "name": "H24 Multiservice - Idraulici H24 a Roma",
      "categories": ["idraulici e lattonieri"],
      "address": { "street": "Via Arimondi Giuseppe, 21", "postal_code": "00159", "city": "Roma", "province": "RM" },
      "phones": ["393 423261391", "340 1054226"],
      "url": "https://www.paginegialle.it/h24multiserviceidraulicih24aroma"
    }
  ],
  "stats": { "requested": 3, "enriched": 0, "with_email": 0, "with_website": 0 }
}
```

Note: with `--enrich=false`, `email`/`website`/`vat` are omitted.

## jq recipes

**Names and phones as TSV:**

```bash
./scripts/paginegialle search --what ristoranti --where torino \
  | jq -r '.items[] | [.name, (.phones|join(", "))] | @tsv'
```

**Full outreach table (contacts, enriched):**

```bash
./scripts/paginegialle contacts --what "studi legali" --where milano --max 30 --delay 1s \
  | jq -r '.items[] | [.name, (.phones|join(", ")), (.email // "-"), (.website // "-"), (.vat // "-")] | @tsv'
```

**Only records that actually have an email:**

```bash
./scripts/paginegialle contacts --what idraulici --where roma --max 50 --delay 1s \
  | jq -r '.items[] | select(.email != null) | [.name, .email] | @tsv'
```

**Coverage summary:**

```bash
./scripts/paginegialle contacts --what idraulici --where roma --max 50 --delay 1s \
  | jq '.stats'
```

**Extract the first URL, then enrich it:**

```bash
slug=$(./scripts/paginegialle search --what "centro estetico" --where milano --fields url \
  | jq -r '.items[0].url')
./scripts/paginegialle get "$slug"
```

**Iterate pages until exhausted:**

```bash
page=1
while :; do
  json=$(./scripts/paginegialle search --what ristoranti --where torino --page "$page" --delay 1s)
  echo "$json" | jq -r '.items[] | [.name, .phones[0]] | @tsv'
  echo "$json" | jq -e '.pagination.has_more' >/dev/null || break
  page=$(echo "$json" | jq -r '.pagination.next_page')
done
```

## Human-readable output

For eyeballing only — do not parse it:

```bash
./scripts/paginegialle search --what idraulici --where roma --human
```
