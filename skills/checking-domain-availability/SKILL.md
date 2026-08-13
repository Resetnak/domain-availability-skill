---
name: checking-domain-availability
description: Use when the user wants to know if a domain name is available or already registered, wants to compare a name across multiple TLDs (.com, .io, .ai, .dev, etc.), or is picking a name for a project/startup/app and needs to see which extensions are free.
---

# Checking Domain Availability

## Overview

Calls the free, keyless domainee.dev API to check whether `name.tld` is registered, for one or many TLDs in a single request.

## API

```
GET https://domainee.dev/api/v1/tools/domain-availability-checker?name=<name>&tlds=<tld>&tlds=<tld>...
```

- `name` (required) — domain name without TLD, e.g. `mybrand`.
- `tlds` (optional, repeatable) — e.g. `?tlds=com&tlds=io`. Omitted → defaults to `com, io, dev, app, co, net, org, me, ai, xyz`.
- No auth needed. **Always use host `domainee.dev/api/...`, not `api.domainee.dev`** — the docs page mentions that host but it 401s ("Missing Bearer token"); the working, CORS-enabled endpoint is same-origin under `domainee.dev/api`.

Rate limit: 30 req/min, 500 req/day per IP. Remaining quota is in `x-ratelimit-remaining-minute` / `x-ratelimit-remaining-day` response headers. A 429 includes `Retry-After` — surface that to the user instead of retrying silently.

Success:
```json
{"ok":true,"data":{"name":"google","results":[
  {"tld":"com","fqdn":"google.com","available":false,"registrarHint":null},
  {"tld":"io","fqdn":"google.io","available":true,"registrarHint":null}
]}}
```
Error (e.g. missing `name`):
```json
{"ok":false,"error":{"code":"missing_param","message":"Query parameter `name` is required (e.g. ?name=mybrand)"}}
```
An unrecognized TLD is **not** an error — it just comes back with an empty `results` array.

A per-TLD registry lookup can also time out. That TLD's entry then has `"available":null` and a `registrarHint` explaining why (e.g. `"The operation was aborted due to timeout"`), while `ok` stays `true` and the other TLDs in the same response are unaffected. Treat `null` as **unknown**, not as available or taken — show it as a separate "couldn't check" row rather than guessing.

## Workflow

1. Strip any TLD the user already typed to get the bare `name`.
2. Call the API. If the user didn't name specific TLDs, omit `tlds` to get the 10-extension default set.
3. Show results as a table: TLD → available (✅) / taken (❌) / unknown-timed out (⚠️, from `registrarHint`).
4. Ask once whether to check more extensions (suggest a few not already covered, e.g. `.ai`, `.so`, a ccTLD) — don't re-ask for the same domain unless the user brings it up again.
5. If they name more TLDs, repeat the call with just those `tlds` and show the new results.

## Example

```bash
curl -s "https://domainee.dev/api/v1/tools/domain-availability-checker?name=mybrand&tlds=com&tlds=io&tlds=ai"
```

## Common Mistakes

- Using `api.domainee.dev` — wrong host, returns 401. Use `domainee.dev/api/...`.
- Treating an empty `results` array as failure — it means the TLD wasn't recognized, not an error.
- Retrying immediately after 429 — respect `Retry-After` and tell the user, don't loop.
