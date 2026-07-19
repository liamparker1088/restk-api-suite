---
id: "337171784058978304"
kind: quickstart
order: 2
createdAt: 1784455217
---
# Quickstart

Get the collection running in a few minutes.

## 1. Point at a test server

Every request uses `{{base_url}}`. Open the environment selector and confirm it resolves to a reachable Restk test server. Nothing in this collection will work until it does.

## 2. Verify connectivity

Open **4. Status Codes → 200 OK** and hit Send. A 200 response with a small JSON body means the server is up and you are ready to go.

If this one fails, stop here and fix the base URL — every other result in the collection is meaningless while the server is unreachable.

## 3. Run a folder

Pick a folder that matches what you want to verify and run it end to end. **1. Methods** is a good starting point: it covers every common verb and finishes with a deliberate 405 case.

Watch the Tests panel rather than just the status code — each request carries a post-response script that asserts the specific protocol behaviour being checked.

## 4. Run the whole collection

Open the Runner tab and execute all 43 requests. Expect some non-2xx responses: the 405, 400, 404, 418, 500, 429 and "Bearer — missing" requests are negative tests that pass by returning an error.

## Configuring auth

The requests in **10. Auth (headline)** need credentials before they will pass:

- **Bearer — valid** needs a token on the Auth tab or in the environment
- **Basic — valid** needs a username and password
- **API key (header)** needs a key and the header name the server expects

**Bearer — missing** deliberately has no credentials — leave it that way, since its 401 is the pass condition.

## A note on multipart

The **multipart form-data** request needs its body fields added manually after import. Until you add them, its parts assertion will stay red. This is a known limitation of `.restk` import, not a failure of the client.
