<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Google Maps API

[![SocQ Google Maps API](https://img.shields.io/badge/API-SocQ%20Google%20Maps%20API-F64C31)](https://socq.ai/apis/google-maps?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/google-maps/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/google-maps-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/google-maps-api/actions/workflows/check.yml)

3 endpoints for public Google Maps data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Google Maps APIs](https://socq.ai/apis/google-maps?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api) · [Documentation](https://docs.socq.ai/api-manual/google-maps/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Google Maps Search API](#google-maps-search-api) | `google-maps/search` | Search public Google Maps places by query and location, returning normalized place identity, contact details, ratings, addresses, and geographic coordinates when available. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Google Maps Place Details API](#google-maps-place-details-api) | `google-maps/place-details` | Collect details for known public Google Maps URLs, returning normalized place identity, category, address, contact, rating, review-count, website, and geographic fields. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Google Maps Reviews API](#google-maps-reviews-api) | `google-maps/reviews` | Collect public reviews from known Google Maps URLs with normalized text, reviewer references, ratings, source links, timestamps, place references, and available media. | 0.15 credits ($0.00075) per result | 2026-08-15 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/google-maps-api.git
cd google-maps-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs google-maps/search

# Python 3.8+ (standard library only)
python3 examples/python/socq.py google-maps/search payloads/search.json

# cURL + jq
bash examples/curl/socq.sh google-maps/search '{"query":"coffee","location":"New York, NY","country":"US","results_limit":3}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Google Maps Search API

Search public Google Maps places by query and location, returning normalized place identity, contact details, ratings, addresses, and geographic coordinates when available.

- **Request:** `POST https://api.socq.ai/v1/google-maps/search`
- **publicId:** `google-maps/search`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/search.json`](payloads/search.json)

Fetch normalized search data from Google Maps.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `country` | string | No | - |
| `latitude` | number | No | (value -90 to 90) |
| `location` | string | Yes | - |
| `longitude` | number | No | (value -180 to 180) |
| `query` | string | Yes | Search query. |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `zoom_level` | integer | No | (value 0 to 21) |

```bash
curl -X POST "https://api.socq.ai/v1/google-maps/search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "coffee",
  "location": "New York, NY",
  "country": "US",
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-maps/search](https://docs.socq.ai/api-manual/google-maps/search) · Landing page: [socq.ai/apis/google-maps/search](https://socq.ai/apis/google-maps/search?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api)

### Google Maps Place Details API

Collect details for known public Google Maps URLs, returning normalized place identity, category, address, contact, rating, review-count, website, and geographic fields.

- **Request:** `POST https://api.socq.ai/v1/google-maps/place-details`
- **publicId:** `google-maps/place-details`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/place-details.json`](payloads/place-details.json)

Fetch normalized places data from Google Maps.

Credits are reserved for each non-empty submitted URL. The task is billed by successfully returned results, and unused reserved credits are refunded.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | A non-empty list of references whose parsed hostname is google.com, goo.gl, or a subdomain of either host. Any scheme is accepted when an authority and hostname are present; protocol-relative references are also accepted, and no specific path is required. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/google-maps/place-details" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.google.com/maps/place/Coffee%20Project%20New%20York%20%7C%20East%20Village/data=!4m7!3m6!1s0x89c2599b5a24d7fd:0x9e354f6cf514b9fc!8m2!3d40.7270884!4d-73.9893820!16s%2Fg%2F11c3svpqld!19sChIJ_dckWptZwokR_LkU9WxPNZ4?authuser=0&hl=en&rclk=1"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-maps/place-details](https://docs.socq.ai/api-manual/google-maps/place-details) · Landing page: [socq.ai/apis/google-maps/place-details](https://socq.ai/apis/google-maps/place-details?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api)

### Google Maps Reviews API

Collect public reviews from known Google Maps URLs with normalized text, reviewer references, ratings, source links, timestamps, place references, and available media.

- **Request:** `POST https://api.socq.ai/v1/google-maps/reviews`
- **publicId:** `google-maps/reviews`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-15
- **Example body:** [`payloads/reviews.json`](payloads/reviews.json)

Fetch normalized reviews data from Google Maps.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `days_limit` | integer | No | (value at least 1) |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `urls` | array of string | Yes | A non-empty list of references whose parsed hostname is google.com, goo.gl, or a subdomain of either host. Any scheme is accepted when an authority and hostname are present; protocol-relative references are also accepted, and no specific path is required. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/google-maps/reviews" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.google.com/maps/place/Coffee%20Project%20New%20York%20%7C%20East%20Village/data=!4m7!3m6!1s0x89c2599b5a24d7fd:0x9e354f6cf514b9fc!8m2!3d40.7270884!4d-73.9893820!16s%2Fg%2F11c3svpqld!19sChIJ_dckWptZwokR_LkU9WxPNZ4?authuser=0&hl=en&rclk=1"
  ],
  "days_limit": 3650,
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-maps/reviews](https://docs.socq.ai/api-manual/google-maps/reviews) · Landing page: [socq.ai/apis/google-maps/reviews](https://socq.ai/apis/google-maps/reviews?utm_source=github&utm_medium=readme&utm_campaign=socq-google-maps-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Google Maps data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
