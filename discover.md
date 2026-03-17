# AI Discover

Send a URL and describe what you want — Iney figures out the selectors, extracts the data, and returns a reusable recipe.

No CSS selectors. No DevTools. Just natural language.

## Quick example

```bash
curl -X POST https://api.iney.io/v1/discover \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://github.com/trending",
    "intent": "trending repos with name and description"
  }'
```

Response:

```json
{
  "status": "success",
  "summary": "GitHub Trending page showing popular repositories sorted by stars gained today. Each entry includes repo name, description, language, and star count.",
  "data": {
    "items": [
      {"name": "obra / superpowers", "description": "An agentic skills framework..."},
      {"name": "codecrafters-io / build-your-own-x", "description": "Master programming..."}
    ]
  },
  "recipe": {
    "id": "recipe_abc123",
    "scrape_request": {
      "extract": {
        "items": {
          "_selector": "article.Box-row[]",
          "_fields": {
            "name": "h2.h3.lh-condensed a.Link",
            "description": "p.col-9.color-fg-muted.my-1.tmp-pr-4"
          }
        }
      }
    },
    "cache_hit": false
  },
  "discover_meta": {
    "phases": {"probe_ms": 576, "llm_ms": 1591, "execute_ms": 3434},
    "total_ms": 5605,
    "llm_model": "qwen-3-235b",
    "cache_hit": false
  }
}
```

## How it works

```
Your request: URL + "what I want"
         │
         ▼
   1. PROBE — fetch the page (HTTP or browser, auto-detected)
   2. ANALYZE — compress HTML to skeleton, send to LLM
   3. EXECUTE — run generated CSS selectors, extract data
         │
         ▼
Response: data + recipe + page summary
```

On repeat requests, the recipe is served from cache — probe and LLM are skipped, only execute runs.

## Request

```
POST /v1/discover
Content-Type: application/json
X-API-Key: YOUR_API_KEY
```

```json
{
  "url": "https://example.com/products",
  "intent": "all products with name and price",

  "hints": {
    "needs_js": true,
    "language": "ru"
  },

  "options": {
    "summary": true,
    "return_recipe": true,
    "save_recipe": true,
    "stealth": false
  }
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `url` | Yes | Page URL to analyze |
| `intent` | Yes | Natural language description of what data you want |
| `hints.needs_js` | No | Force browser rendering (skip HTTP probe) |
| `hints.language` | No | Content language hint for LLM (e.g. `"ru"`) |
| `options.summary` | No | Include page summary (default: `true`) |
| `options.return_recipe` | No | Include recipe in response (default: `true`) |
| `options.save_recipe` | No | Cache recipe for future requests (default: `true`) |
| `options.stealth` | No | Enable anti-bot stealth mode |

## Response

### Success

| Field | Description |
|-------|-------------|
| `summary` | 2-3 sentence page description (English) |
| `data` | Extracted data matching your intent |
| `data_meta.items_found` | Number of items extracted |
| `data_meta.confidence` | LLM confidence score (0.0-1.0) |
| `recipe.id` | Recipe ID for reference |
| `recipe.scrape_request` | Reusable config for `/v1/scrape` |
| `recipe.cache_hit` | Whether recipe came from cache |
| `discover_meta.phases` | Per-phase timing (probe, llm, execute) |
| `discover_meta.total_ms` | Total processing time |

### Errors

| Code | HTTP | Description |
|------|------|-------------|
| `DISCOVER_PROBE_BLOCKED` | 422 | Target blocked the request. Try `hints.needs_js=true` |
| `DISCOVER_NO_STRUCTURE` | 422 | Could not extract data. Try a more specific intent |
| `DISCOVER_LOW_CONFIDENCE` | 422 | Low confidence in result. Partial recipe included |
| `DISCOVER_LLM_TIMEOUT` | 503 | LLM timed out. Retry in a few seconds |

## Reuse the recipe

The `recipe.scrape_request` field is a ready-to-use `/v1/scrape` request body. Use it for subsequent calls without paying for LLM analysis:

```bash
curl -X POST https://api.iney.io/v1/scrape \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://github.com/trending",
    "extract": {
      "items": {
        "_selector": "article.Box-row[]",
        "_fields": {
          "name": "h2.h3.lh-condensed a.Link",
          "description": "p.col-9.color-fg-muted.my-1.tmp-pr-4"
        }
      }
    }
  }'
```

Cost: 1 unit (vs 12 for discover).

## Page summary for context

Every discover response includes a `summary` field — a short English description of the page. Useful when you need page context without structured extraction:

```json
{
  "summary": "A Russian-language job listings page on hh.ru showing software developer positions in Moscow. Each listing includes job title, company name, salary range, and requirements."
}
```

Disable with `"options": {"summary": false}`.

## Unit costs

| Scenario | Cost |
|----------|------|
| Cache miss, static site | 12 units |
| Cache miss, JS site | 30 units |
| Cache miss, auto-upgrade (HTTP→browser) | 31 units |
| Cache hit | 1-10 units |
| Reuse recipe via `/v1/scrape` | 1-10 units |

## Qarap integration

When a user shares a link in a conversation, use `/v1/discover` to get both data and context:

```python
response = requests.post("https://api.iney.io/v1/discover", json={
    "url": user_link,
    "intent": user_message,
    "options": {"summary": True}
}, headers={"X-API-Key": INEY_KEY})

result = response.json()
page_context = result.get("summary", "")
extracted_data = result.get("data", {})

# Feed both to the bot for a rich, informed response
```

The summary provides page-level context. The extracted data provides structured facts. Together they give the bot everything it needs to answer intelligently about the linked page.
