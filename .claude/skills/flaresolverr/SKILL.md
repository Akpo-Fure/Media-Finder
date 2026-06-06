---
name: flaresolverr
description: >-
  Understand or fix FlareSolverr in the Media Finder stack (port 8191) — the background helper that
  gets past Cloudflare on protected torrent indexers. Use when a Prowlarr indexer fails with a
  Cloudflare challenge.
---

# FlareSolverr (port 8191) — Cloudflare helper

FlareSolverr runs in the background. It has **no web UI and no setup of its own** — it only matters
through Prowlarr.

## How it's used
- In **Prowlarr**: Settings → Indexers → Indexer Proxies → + → FlareSolverr, Host
  `http://flaresolverr:8191`, with a Tag (e.g. `flaresolverr`).
- Apply that **tag only to indexers that are Cloudflare-protected**. A FlareSolverr proxy only acts
  on indexers that share its tag. **Most indexers need no tag at all** — leave the Tags box blank
  unless a site fails with a Cloudflare challenge (e.g. Nyaa, AnimeTosho, YTS need no tag).

## Gotchas
- An indexer that fails with a Cloudflare / challenge error usually just needs the `flaresolverr`
  tag added to it.
- A **"451"** error is NOT Cloudflare — that means the site is blocked or closed (see the `prowlarr`
  skill); FlareSolverr won't help with that.

## Check
`docker logs flaresolverr` (should show it ready) ·
`Invoke-WebRequest http://localhost:8191 -UseBasicParsing`.
