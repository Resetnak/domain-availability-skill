---
name: checking-domain-availability
description: Use when the user wants to know if a domain name is available or already registered, wants to compare a name across multiple TLDs (.com, .io, .ai, .dev, etc.), or is naming a project/startup/app and wants to check the domain plus whether the matching npm package name and GitHub username are free too.
---

# Checking Domain Availability

## Overview

Checks whether a `name` is free across three surfaces a project-naming decision usually depends on: domain TLDs, npm package name, and GitHub username/org. Each check is a separate free, keyless API call — run them together when the user is naming something, or just the domain check when that's all they asked for.

## Domain API

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

A per-TLD registry lookup can also time out. That TLD's entry then has `"available":null` and a `registrarHint` explaining why (e.g. `"The operation was aborted due to timeout"`), while `ok` stays `true` and the other TLDs in the same response are unaffected. Treat `null` as **unknown**, not as available or taken — show it as a separate "couldn't check" row rather than guessing. If *every* TLD in the response comes back `null` at once, that's a transient hiccup on the whole request, not 10 coincidental timeouts — retry the call once before reporting an all-unknown result to the user.

## npm package name API

```
GET https://registry.npmjs.org/<name>
```

No auth, no rate-limit headers to worry about in practice. `200` = name taken (a real package exists), `404` = free. For scoped names (`@scope/name`), URL-encode the slash: `https://registry.npmjs.org/@scope%2Fname`.

## GitHub username/org API

```
GET https://api.github.com/users/<name>
```

No auth needed. `200` = taken, `404` = free. Unauthenticated requests are capped at 60/hour per IP (`x-ratelimit-remaining` header) — fine for a handful of checks, but don't loop this over a big list of candidate names.

## Workflow

1. Strip any TLD the user already typed to get the bare `name`.
2. If the user is just asking about a domain, only call the domain API. If they're naming a project/startup/package/tool, run all three checks (domain, npm, GitHub) for that one `name` — this combo is the actual point of the skill, not domains in isolation.
3. Call the domain API. If the user didn't name specific TLDs, omit `tlds` to get the 10-extension default set.
4. Show one combined summary: a TLD table (available ✅ / taken ❌ / unknown-timed out ⚠️, from `registrarHint`), plus one line each for npm and GitHub (✅/❌).
5. Ask once whether to check more TLDs (suggest a few not already covered, e.g. `.ai`, `.so`, a ccTLD) — don't re-ask for the same name unless the user brings it up again.
6. If they name more TLDs, repeat the domain call with just those `tlds` and show the new results.

Social handles (X, Instagram, etc.) are deliberately **not** included — none of those platforms expose a free, unauthenticated availability check; scraping profile pages is unreliable and against most of their terms. Say so if asked, rather than guessing from a scrape.

## Example

```bash
curl -s "https://domainee.dev/api/v1/tools/domain-availability-checker?name=mybrand&tlds=com&tlds=io&tlds=ai"
curl -s -o /dev/null -w "%{http_code}" "https://registry.npmjs.org/mybrand"
curl -s -o /dev/null -w "%{http_code}" "https://api.github.com/users/mybrand"
```

## Common Mistakes

- Using `api.domainee.dev` — wrong host, returns 401. Use `domainee.dev/api/...`.
- Treating an empty `results` array as failure — it means the TLD wasn't recognized, not an error.
- Retrying immediately after 429 — respect `Retry-After` and tell the user, don't loop.
- Checking npm/GitHub for a plain "is this domain free?" question — only pull those in when the user is actually naming something, not for every domain lookup.
- Claiming to check social handles — there's no reliable free API for that; say it's out of scope.
