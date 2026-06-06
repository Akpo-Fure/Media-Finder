---
name: radarr
description: >-
  Set up or fix Radarr (movies) in the Media Finder stack — web UI on http://localhost:7878. Use to
  set the movie root folder, connect qBittorrent, choose quality, wire Plex notifications, or fix
  "Unable to connect to qBittorrent" / "Authentication Failure".
---

# Radarr (port 7878) — movies

Web UI: http://localhost:7878. Set a login when asked.

## Setup
1. **Root folder:** Settings → Media Management → Root Folders → + → `/videos/Movies`.
2. **Download client:** Settings → Download Clients → + → qBittorrent. Host **`qbittorrent`**
   (NOT `localhost`), Port `8080`, Category `movies`. Test → Save.
3. **Quality:** Settings → Profiles → Quality Profiles → use **HD-1080p** (avoid 4K — huge and
   slow, with few seeders).
4. **Plex notifications:** Settings → Connect → + → Plex Media Server. Host `host.docker.internal`,
   Port `32400`, **Authenticate with Plex.tv**. (If Test fails, use the PC's LAN IPv4 from
   `ipconfig`.)

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
