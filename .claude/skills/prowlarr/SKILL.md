---
name: prowlarr
description: >-
  Set up or fix Prowlarr (the indexer / search-site manager) in the Media Finder stack — web UI on
  http://localhost:9696. Use to add torrent indexers, wire FlareSolverr, link Radarr/Sonarr so
  indexers sync automatically, or diagnose indexers that won't connect.
---

# Prowlarr (port 9696) — search-site manager

Web UI: http://localhost:9696. Set a login on first open.

## Setup
1. **Add FlareSolverr** (for Cloudflare-protected sites): Settings → Indexers → Indexer Proxies →
   + → FlareSolverr. Host `http://flaresolverr:8191`, add a Tag `flaresolverr`. Test → Save.
2. **Add indexers:** Indexers → Add Indexer. Working public ones: **Knaben, The Pirate Bay, 1337x,
   YTS** (YTS is movies-only). Test each.
   - **Tags:** leave the Tags box **blank** for normal indexers (Knaben, 1337x, YTS, Nyaa, etc.).
     Only when a site fails with a **Cloudflare** challenge do you open it and add the
     `flaresolverr` tag.
   - **Anime:** also add **Nyaa** and **AnimeTosho** (the main anime trackers, usually pre-subbed).
   - **K-drama:** the public trackers above already cover popular titles — no special indexer needed.
     Deep/old catalogs need private Asian trackers (e.g. AvistaZ, account required).
3. **Link Radarr + Sonarr** (do this after those apps exist): Settings → Apps → + → Radarr.
   Prowlarr Server `http://prowlarr:9696`, Radarr Server `http://radarr:7878`, API Key from
   Radarr → Settings → General, Sync Level **Full Sync**. Repeat for Sonarr (`http://sonarr:8989`).
   Indexers then appear in both automatically.

## Gotchas
- **Dead sites:** TorrentGalaxy (shut down) and EZTV (returns "451") will just error — skip them.
- **Almost everything returns "451" / redirects** → the ISP is blocking torrent sites → enable the
  VPN (see the `media-finder-setup` skill / README VPN section).
- Movies-only indexers (YTS) sync to Radarr only — that is correct.

## Check
After linking, confirm the sync (API key from `C:\MediaStack\config\radarr\config.xml`):
`Invoke-RestMethod http://localhost:7878/api/v3/indexer -Headers @{"X-Api-Key"="<radarr key>"}`.
