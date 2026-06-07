---
name: sonarr
description: >-
  Set up or fix Sonarr (TV + anime) in the Media Finder stack — web UI on http://localhost:8989. Use
  to set the TV/anime root folders, connect qBittorrent, choose quality, wire Plex + Jellyfin
  notifications, and handle anime (Series Type, numbering, release tuning). Same pattern as Radarr.
---

# Sonarr (port 8989) — TV & anime

Web UI: http://localhost:8989. Same pattern as Radarr, with TV values.

## Setup
1. **Root folders:** Settings → Media Management → Root Folders → add **`/videos/TV Shows`** and
   **`/videos/Anime`** (anime gets its own folder because it uses absolute episode numbering).
2. **Download client → qBittorrent:** Host **`qbittorrent`** (NOT `localhost`), Port `8080`,
   Category `tv`. Test → Save. Tick **Remove Completed** (Completed Download Handling) so a finished
   torrent is cleared once imported — downloads and the library are separate copies, so this reclaims
   the duplicate disk space.
3. **Quality:** Quality Profile **HD-1080p** (avoid 4K — huge, slow, few seeders). Add a Release
   Profile (Settings → Profiles → Release Profiles) with **Ignored Term `AV1`** so releases come as
   x264/x265 — AV1 won't play on many phones/TVs and forces CPU transcoding (no GPU under Docker
   Desktop).
4. **Plex + Jellyfin notifications:** Settings → Connect → +. Add **Plex Media Server** (host
   `host.docker.internal`, port `32400`, Authenticate with Plex.tv) and **Emby / Jellyfin** (host
   `jellyfin`, port `8096`, API key from Jellyfin → Dashboard → API Keys, tick **Update Library**).
   Both refresh the player the moment an episode imports.
5. **Anime:** an anime show's **Series Type = Anime** (handles absolute numbering); Seerr sets this
   automatically via its anime fields (see the `seerr` skill). Anime needs **Nyaa** / **AnimeTosho**
   in Prowlarr (see the `prowlarr` skill) and the Jellyfin Anime library on English metadata (see the
   `jellyfin` skill).

## Gotchas / fixes
- Download client Host = `qbittorrent`, not `localhost`. "Authentication Failure" → qBittorrent
  whitelist (see the `qbittorrent` skill).
- **Anime over-grabbing:** requesting a long-running anime makes Sonarr search every episode at once
  and grab several overlapping releases (season packs, complete-series batches, per-episode, different
  fansub groups) — some stall or duplicate. Pick one well-seeded batch via **Interactive Search**, and
  remove the extras from qBittorrent.
- **Anime won't import ("Episode X was not found in the grabbed release"):** a batch's anime numbering
  doesn't line up with Sonarr's episode list. Recover via **Activity → Manual Import** (also under
  Wanted): point it at the download folder and Sonarr re-reads each file by `SxxExx` and maps it. A
  manual import doesn't fire the Plex/Jellyfin notifications, so scan those libraries by hand after.
- **Language Profile:** if Sonarr or Seerr asks for one, pick **English** (or the only option).
- Sonarr auto-grabs **new episodes** of monitored shows as they air.

## API key
Settings → General, or `C:\MediaStack\config\sonarr\config.xml`.

## Check
`Invoke-RestMethod http://localhost:8989/api/v3/queue?pageSize=50 -Headers @{"X-Api-Key"="<key>"}`.
