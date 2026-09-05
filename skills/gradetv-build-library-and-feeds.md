---
name: Build a personal library and export it as M3U feeds
description: Mint a guest identity, organize channels into folders and groups, and hand back stable feed URLs playable in VLC.
api: openapi/gradetv-openapi.json
operations: [create_guest, mint_key, create_category, create_group, add_item, get_library]
generated: '2026-09-05'
method: generated
---

# Build a library and get its feeds

The guest token IS the identity — everything the user builds hangs off it.

1. **Mint identity** — `create_guest` (`POST /api/guest`, no auth) returns an
   `ipt_...` token. Persist it: losing it loses the library. For a durable
   agent credential, `mint_key` (`POST /api/keys`, guest auth) mints an
   `iptk_...` API key whose full text appears exactly once.
2. **Authenticate** — send the guest token or API key as a bearer token
   (`Authorization: Bearer ipt_...`).
3. **Create structure** — `create_category` (`POST /api/categories`) makes a
   folder (a "Geral" group is created inside automatically); `create_group`
   (`POST /api/groups`) adds a sub-tab. Caps from `GET /api/billing`:
   8 categories, 12 groups per category, 40 items per group (exceeding the
   item cap is a 400).
4. **Add channels** — `add_item` (`POST /api/items`) puts a catalog channel
   into a group.
5. **Read it back** — `get_library` (`GET /api/library`) returns the whole
   tree WITH feed URLs at every level: whole library
   (`GET /f/{token}/library.{formato}`), per folder, per group — formats
   m3u, m3u8, json, xspf, ready for VLC or the extension.

Reversals exist for every step (delete item/group/category, revoke key) but no
undo windows are published — deletes are immediate and cascade
(see conventions/gradetv-conventions.yml reversibility block).

If the user later logs in by email (`auth_start` + `auth_verify`), verify
returns the account's ORIGINAL guest (`claimed.product.guest_token`,
`adopt: true`) — switch to that token; it owns the existing library.
