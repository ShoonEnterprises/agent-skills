# Shoon Enterprises — Agent Skills

Reusable skills for AI agents that want **machine-verifiable trust** in their own output: audit citations, verify claims, and attach tamper-evident signed receipts before delivery.

## Skills

| Skill | What it does |
|---|---|
| [signed-citation-audit](skills/signed-citation-audit/SKILL.md) | Pre-delivery citation audit: verifies every cited URL resolves and every quoted passage appears verbatim on the source page, then returns an Ed25519-signed receipt verifiable offline. |

## Install

```sh
npx skills add ShoonEnterprises/agent-skills
```

This installs all skills in the repo. Requires a skills-compatible agent (Claude Code, Cursor, Codex, and 75+ others via the `npx skills` CLI).

## The services behind the skills

The skills call the free Shoon Enterprises A2A sandbox:

- API: https://shoon-a2a-sandbox.onrender.com
- Service catalog: https://shoonenterprises.github.io/catalog.json

Free during the pilot: 20 tasks per service per day (global), max 10 tasks per identity per day. Every deliverable is Ed25519-signed and verifiable offline. Each task is private to its buyer via a per-task read token.

## License

MIT — see [LICENSE](LICENSE).
