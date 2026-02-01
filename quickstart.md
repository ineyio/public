# Quick Start

Iney API works without registration. Send a POST request and get the result.

## First request

```bash
curl -X POST https://api.iney.io/v1/scrape \
  -H "Content-Type: application/json" \
  -d '{"url": "https://quotes.toscrape.com"}'
```

Response:

```json
{
  "status_code": 200,
  "body": "<html>...</html>",
  "meta": {
    "request_id": "req_abc123",
    "processing_ms": 450,
    "cost_units": 1
  }
}
```

## Browser rendering

Add `"javascript": true` for pages that require JS.

```bash
curl -X POST https://api.iney.io/v1/scrape \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://quotes.toscrape.com/js",
    "javascript": true
  }'
```

Cost: 10 units (1 base + 9 browser).

## Data extraction

Extract structured data with CSS selectors.

```bash
curl -X POST https://api.iney.io/v1/scrape \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://quotes.toscrape.com",
    "extract": {
      "quotes": {
        "_selector": ".quote[]",
        "_fields": {
          "text": ".text",
          "author": ".author"
        }
      }
    }
  }'
```

Response includes `extracted` field with structured data:

```json
{
  "status_code": 200,
  "extracted": {
    "quotes": [
      {"text": "The world as we have...", "author": "Albert Einstein"},
      {"text": "It is our choices...", "author": "J.K. Rowling"}
    ]
  }
}
```

## Screenshot

```bash
curl -X POST https://api.iney.io/v1/scrape \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "javascript": true,
    "screenshot": true
  }'
```

The response will contain `screenshot_url` with a link to the JPEG image.

## Check your quota

```bash
curl https://api.iney.io/v1/anonymous/usage
```

```json
{
  "tier": "anonymous",
  "used": 11,
  "quota": 50,
  "remaining": 39,
  "period": "daily",
  "reset_at": "2026-02-02T00:00:00Z"
}
```

Anonymous tier: 50 units/day, resets at 00:00 UTC. No registration required.

Need more? [Register for Free tier](/docs/tiers) — 200 units/day and 3 concurrent requests.
