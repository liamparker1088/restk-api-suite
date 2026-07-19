---
id: "337171746582872064"
kind: welcome
order: 1
createdAt: 1784455208
---
# Restk HTTP Protocol Coverage

A conformance suite that exercises the HTTP surface an API client is expected to handle correctly — verbs, bodies, headers, status codes, redirects, compression, cookies, timing, payload rendering and authentication.

Every request targets the Restk test server via the `{{base_url}}` variable, and each carries a post-response script that asserts the expected behaviour. Run the whole collection to get a pass/fail picture of protocol handling in one sweep.

## How it is organised

The 43 requests sit in ten folders, each covering one axis of the protocol:

| Folder | Covers |
|---|---|
| 1. Methods | GET, POST, PUT, PATCH, DELETE, OPTIONS, plus a 405 mismatch case |
| 2. Body Types | raw JSON, raw XML, URL-encoded, multipart, content-type enforcement |
| 3. Headers & Query | automatic headers, user overrides, response headers |
| 4. Status Codes | 200, 201, 204, 400, 404, 418, 500 |
| 5. Redirects | multi-hop chains, 307 preserve, 302 downgrade |
| 6. Content-Encoding | gzip, deflate, brotli |
| 7. Cookies | persistence across calls, attribute parsing |
| 8. Timeouts & Retries | latency measurement, 429 rate limiting |
| 9. Payload & Render | binary, NDJSON streams, text, HTML, images, mismatched types |
| 10. Auth (headline) | Bearer, Basic and API key |

## Reading the results

Several requests are **negative tests** — they pass by returning an error. The 405, 400, 404, 418, 500, 429 and "Bearer — missing" requests are all expected to return non-2xx codes. A green result means the client surfaced the error correctly, not that the call succeeded.

## Before you run

Point `{{base_url}}` at a running Restk test server. If "200 OK" in the Status Codes folder fails, the server is unreachable and every other result in the collection is unreliable.
