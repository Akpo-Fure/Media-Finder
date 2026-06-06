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
1. **Root folder:** Settings → Media Management → Root Folders → + → `/videos/TV Shows`.
2. **Download client → qBittorrent:** Host **`qbittorrent`** (NOT `localhost`), Port `8080`,
   Category `tv`. Test → Save.
3. **Quality:** Quality Profile **HD-1080p**.
4. **Plex notifications:** Settings → Connect → Plex Media Server, host `host.docker.internal`,
   port `32400`, Authenticate with Plex.tv.

## Anime (proper handling)

Anime uses absolute episode numbering, so give it its own library:
1. **Root folder:** add `/videos/Anime` (Settings → Media Management → Root Folders).
2. **Series type:** for an anime show set its **Series Type = Anime** (Sonarr then handles absolute
   numbering and anime release groups). Seerr can do this automatically — see the `seerr` skill's
   anime routing.
3. Make sure **Nyaa** / **AnimeTosho** are added in Prowlarr (see the `prowlarr` skill) so anime has
   a source.
4. Quality: HD-1080p is fine, or create a dedicated "Anime" profile.

## Gotchas / fixes
- Download client Host = `qbittorrent`, not `localhost`. "Authentication Failure" → qBittorrent
  whitelist (see the `qbittorrent` skill).
- If a **Language Profile** is requested (e.g. in Seerr's Sonarr settings), pick the one available.
- Sonarr auto-grabs **new episodes** of monitored shows as they air.

## API key
Settings → General, or `C:\MediaStack\config\sonarr\config.xml`.

## Check
`Invoke-RestMethod http://localhost:8989/api/v3/queue?pageSize=50 -Headers @{"X-Api-Key"="<key>"}`.
