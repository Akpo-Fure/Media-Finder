---
name: sonarr
description: >-
  Set up or fix Sonarr (TV shows) in the Media Finder stack — web UI on http://localhost:8989. Use
  to set the TV root folder, connect qBittorrent, choose quality, and wire Plex notifications. Same
  pattern as Radarr, with TV values.
---

# Sonarr (port 8989) — TV

Web UI: http://localhost:8989. Identical to Radarr, with TV values.

## Setup
1. **Root folders:** Settings → Media Management → Root Folders → add **`/videos/TV Shows`** and
   **`/videos/Anime`** (anime gets its own folder because it uses absolute episode numbering).
2. **Download client → qBittorrent:** Host **`qbittorrent`** (NOT `localhost`), Port `8080`,
   Category `tv`. Test → Save.
3. **Quality:** Quality Profile **HD-1080p**.
4. **Plex notifications:** Settings → Connect → Plex Media Server, host `host.docker.internal`,
   port `32400`, Authenticate with Plex.tv.
5. **Anime:** for an anime show set its **Series Type = Anime** (handles absolute numbering). Seerr
   sets this automatically via its anime fields (see the `seerr` skill); anime needs **Nyaa** /
   **AnimeTosho** in Prowlarr (see the `prowlarr` skill).

## Gotchas / fixes
- Download client Host = `qbittorrent`, not `localhost`. "Authentication Failure" → qBittorrent
  whitelist (see the `qbittorrent` skill).
- If a **Language Profile** is requested (e.g. in Seerr's Sonarr settings), pick the one available.
- Sonarr auto-grabs **new episodes** of monitored shows as they air.

## API key
Settings → General, or `C:\MediaStack\config\sonarr\config.xml`.

## Check
`Invoke-RestMethod http://localhost:8989/api/v3/queue?pageSize=50 -Headers @{"X-Api-Key"="<key>"}`.
