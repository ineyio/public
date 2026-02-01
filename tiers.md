# Tiers

## Anonymous

No registration. Works by IP address.

| Parameter | Value |
|-----------|-------|
| Daily quota | 50 units |
| Concurrency | 1 |
| Max timeout | 30s |
| Browser (JS) | Yes |
| Screenshot | 640x800, quality 30% |
| History retention | 24 hours |
| Authentication | None |

Quota resets daily at 00:00 UTC.

Optional: pass `X-Anon-Session-ID` header to track your requests and view history via `/v1/anonymous/requests`.

## Free

Requires registration and API key.

| Parameter | Value |
|-----------|-------|
| Daily quota | 200 units |
| Concurrency | 3 |
| Max timeout | 60s |
| Browser (JS) | Yes |
| Screenshot | Full page, quality 50% |
| History retention | 30 days |
| Authentication | `X-API-Key` header |

All Anonymous features plus:
- HTML download (`/v1/requests/{id}/html`)
- Content re-extraction (`/v1/requests/{id}/extract`)
- Budget statistics (`/v1/budget/stats`)

## Unit costs

| Operation | Cost |
|-----------|------|
| HTTP request | 1 |
| + Browser (JS) | +9 |
| + Screenshot | +2 |
| + Full resources | +5 |
| + Extraction | +1 |
| + DC proxy (IPv4) | +2 |
| + Retry attempt | +1 |

Examples:
- Simple HTTP scrape: **1 unit**
- Browser render: **10 units**
- Browser + screenshot: **12 units**

## Blocked domains (beta)

During beta, the following domains are blocked for all tiers: google.com, linkedin.com, facebook.com, instagram.com, twitter.com, x.com, amazon.com, cloudflare.com.
