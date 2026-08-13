# domain-availability-skill

**Agent skill: check if a name is free — domain TLDs, npm package, and GitHub username, in one go. No API key required.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![skills.sh](https://skills.sh/b/Resetnak/domain-availability-skill)](https://skills.sh/Resetnak/domain-availability-skill)
![Agent Skill](https://img.shields.io/badge/agent--skill-SKILL.md-blueviolet)
![No API key](https://img.shields.io/badge/API%20key-not%20required-brightgreen)

The tiny problem that eats 20 minutes every time you name something: is the domain free, is the npm name taken, is the GitHub handle available? Ask your coding agent once, get all three.

```
you:   I'm naming a new tool "coolstartup" — is it available?
agent: checking domain, npm, and GitHub...

       .com  ❌ taken       npm     ✅ free
       .io   ✅ available   GitHub  ✅ free
       .dev  ✅ available
       .ai   ✅ available
       ...

       want me to check any other TLDs?
```

## Why

- **Zero setup** — no API key, no config, just the skill.
- **Checks the whole naming decision, not just the domain** — TLDs, npm package name, and GitHub username/org together when you're naming a project; just the domain when that's all you asked. PyPI and crates.io are one ask away if you name that ecosystem.
- **Multi-TLD in one call** — the default set (`com`, `io`, `dev`, `app`, `co`, `net`, `org`, `me`, `ai`, `xyz`) or any TLDs you name.
- **Interactive** — asks once whether to widen the TLD search, doesn't nag.
- **Handles the edge cases** — unrecognized TLDs, per-TLD registry timeouts, and whole-request timeouts are documented and retried sanely, not silently mis-reported.
- **Honest about what it doesn't do** — social handles (X, Instagram, ...) have no free unauthenticated availability API, so the skill says so instead of guessing from a scrape.

## Install

```bash
npx skills add Resetnak/domain-availability-skill
```

Or copy `skills/checking-domain-availability/` straight into your agent's skills directory (e.g. `~/.claude/skills/`).

## How it works

See [`skills/checking-domain-availability/SKILL.md`](skills/checking-domain-availability/SKILL.md) — three free, keyless GET requests:

- Domain TLDs: `domainee.dev/api/v1/tools/domain-availability-checker` (30/min, 500/day per IP)
- npm: `registry.npmjs.org/<name>` (`404` = free)
- GitHub: `api.github.com/users/<name>` (`404` = free, 60/hour unauthenticated)

## License

MIT — see [LICENSE](LICENSE).
