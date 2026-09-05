---
name: Find a live TV or radio channel and check it actually plays
description: Search the Grade public catalog, read a channel's health verdict for your country, get today's guide, and report back whether it played.
api: openapi/gradetv-openapi.json
operations: [search_channels, get_channel, channel_health, get_channel_guide, report_play]
generated: '2026-09-05'
method: generated
---

# Find a channel and check it plays

All catalog reads are free and need no auth. Prefer the MCP server at
`POST https://gradetv.net/mcp` — every tool name below IS the REST operationId.

1. **Search** — `search_channels` (`GET /api/channels`) with filters `q`,
   `country` (ISO 3166-1 alpha-2), `category`, `language` (ISO 639-3),
   `network`, `quality`, `kind` (`tv` default, `radio` for Radio Browser
   stations), `tag` (radio only). Page with `limit`/`offset`; `limit` above 50
   is silently clamped to 50 — continue from `next_offset`. `sort=score` orders
   by measured health; `online=1` keeps only channels seen online.
2. **Inspect** — `get_channel` (`GET /api/channels/{id}`) returns the full
   record with streams already pointing at Grade's counting hop
   (`GET /api/s/{id}`) and `social` counters.
3. **Trust, but verify** — `channel_health` (`GET /api/channels/{id}/health`)
   explains failures by environment and country: `regions[]`, `geo`
   (geo-blocked vs down, with countries), `latency[]`, and `pra_voce` — the
   verdict for the caller's own country. Use it to tell "down" from
   "geo-blocked" from "just slow for you".
4. **Guide** — `get_channel_guide` (`GET /api/channels/{id}/guia`) gives
   today's EPG: now, next, and the day's list.
5. **Close the loop** — after playing, POST `report_play`
   (`POST /api/play-report`) with ok=true or ok=false + failure code. Without
   reports the health catalog does not learn.

Gotchas: adult content is closed by default — `nsfw=1` returns
`403 nsfw_consent_required` until the owner accepts the 18+ terms
(`nsfw_consent_accept`). Errors are a plain JSON envelope with a `code`
field, not RFC 9457 (see conventions/gradetv-conventions.yml).
