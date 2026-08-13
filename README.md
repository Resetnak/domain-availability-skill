# checking-domain-availability

A skill that checks whether a domain name is registered across one or more TLDs, using the free [domainee.dev](https://domainee.dev/docs/free-apis/domain-availability-checker) API. No API key needed.

Ask your agent things like:

- "is coolstartup available as a domain?"
- "check coolstartup.com, and also try .ai and .so"

It reports availability per TLD and offers to check further extensions.

## Install

```bash
npx skillsadd <owner>/domain-availability-skill
```

Or copy `skills/checking-domain-availability/` into your agent's skills directory (e.g. `~/.claude/skills/`).

## License

MIT
