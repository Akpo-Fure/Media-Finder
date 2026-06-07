---
name: radarr
description: >-
  Set up or fix Radarr (movies) in the Media Finder stack — web UI on http://localhost:7878. Use to
  set the movie root folder, connect qBittorrent, choose quality, wire Plex + Jellyfin notifications,
  or fix "Unable to connect to qBittorrent" / "Authentication Failure".
---

# Radarr (port 7878) — movies

Web UI: http://localhost:7878. Set a login when asked.

## Setup
1. **Root folder:** Settings → Media Management → Root Folders → + → `/videos/Movies`.
2. **Download client:** Settings → Download Clients → + → qBittorrent. Host **`qbittorrent`**
   (NOT `localhost`), Port `8080`, Category `movies`. Test → Save. Tick **Remove Completed**
   (Completed Download Handling) so a finished torrent is cleared once imported — downloads and the
   library are separate copies, so this reclaims the duplicate disk space.
3. **Quality:** Settings → Profiles → Quality Profiles → use **HD-1080p** (avoid 4K — huge and
   slow, with few seeders).
4. **Plex + Jellyfin notifications:** Settings → Connect → +. Add **Plex Media Server** (host
   `host.docker.internal`, port `32400`, **Authenticate with Plex.tv**; if Test fails, use the PC's
   LAN IPv4 from `ipconfig`) and **Emby / Jellyfin** (host `jellyfin`, port `8096`, API key from
   Jellyfin → Dashboard → API Keys, tick **Update Library**). Both refresh the player the moment a
   movie imports.

## Gotchas / fixes
- **"Unable to connect to qBittorrent"** → Host must be `qbittorrent`, not `localhost`.
- **"Authentication Failure"** → set the subnet whitelist in qBittorrent (see the `qbittorrent`
  skill); then the username/password no longer matter.
- Downloads taking "days" → it grabbed a 4K release; set 1080p (step 3) or use **Interactive
  Search** and pick a well-seeded 1080p result.

## API key
Settings → General, or `C:\MediaStack\config\radarr\config.xml` (`<ApiKey>`). Needed by Prowlarr,
Seerr, and Bazarr.

## Check
`Invoke-RestMethod http://localhost:7878/api/v3/queue?pageSize=50 -Headers @{"X-Api-Key"="<key>"}`.
