# domain-availability-skill

**Agent skill: check domain name availability across TLDs, no API key required.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![skills.sh](https://skills.sh/b/Resetnak/domain-availability-skill)](https://skills.sh/Resetnak/domain-availability-skill)
![Agent Skill](https://img.shields.io/badge/agent--skill-SKILL.md-blueviolet)
![No API key](https://img.shields.io/badge/API%20key-not%20required-brightgreen)

Ask your coding agent to check whether a domain is free — it calls the free [domainee.dev](https://domainee.dev/docs/free-apis/domain-availability-checker) API, shows availability per TLD, and offers to check further extensions (`.ai`, `.io`, `.so`, ccTLDs, ...).

```
you:   is "coolstartup" available as a domain?
agent: checking the usual extensions...

       .com  ❌ taken
       .io   ✅ available
       .dev  ✅ available
       .ai   ✅ available
       ...

       want me to check any other extensions?
```

## Why

- **Zero setup** — no API key, no config, just the skill.
- **Multi-TLD in one call** — checks up to the whole default set (`com`, `io`, `dev`, `app`, `co`, `net`, `org`, `me`, `ai`, `xyz`) or any TLDs you name.
- **Interactive** — asks once whether to widen the search, doesn't nag.
- **Handles the edge cases** — unrecognized TLDs and per-TLD registry timeouts are documented, not silently mis-reported.

## Install

```bash
npx skills add Resetnak/domain-availability-skill
```

Or copy `skills/checking-domain-availability/` straight into your agent's skills directory (e.g. `~/.claude/skills/`).

## How it works

See [`skills/checking-domain-availability/SKILL.md`](skills/checking-domain-availability/SKILL.md) — one `GET` request to `domainee.dev/api/v1/tools/domain-availability-checker`, rate-limited to 30/min and 500/day per IP.

## License

MIT — see [LICENSE](LICENSE).
