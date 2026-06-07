---
name: seerr
description: >-
  Set up or fix Seerr (the request website, the rebranded Overseerr) in the Media Finder stack —
  web UI on http://localhost:5055. Use to sign in with Plex, connect Plex, add the Radarr/Sonarr
  servers so requests route automatically, or fix Seerr container/permission issues.
---

# Seerr (port 5055) — the request page

Web UI: http://localhost:5055. Seerr is the rebranded successor to Overseerr.

## Container facts (different from the *arr apps)
- Image `ghcr.io/seerr-team/seerr:latest`; runs as the **node** user (UID 1000); config at
  **`/app/config`**; needs **`init: true`**; has **no PUID/PGID**.
- If Seerr cannot write its config on first run:
  `docker run --rm -v C:/MediaStack/config/seerr:/data alpine chown -R 1000:1000 /data`, then
  restart Seerr.

## Setup
1. **Sign in with Plex.**
2. **Connect Plex:** press the button next to the Server dropdown to fetch it from plex.tv, or
   enter Hostname `host.docker.internal`, Port `32400` (or the PC LAN IP). Save → **Sync Libraries**
   → **enable all of them (Movies, TV Shows, Anime)**. Seerr only marks titles **Available** for
   libraries that are enabled here, so leave none off; if you add a library later (e.g. Anime), click
   **Sync Libraries** again to pick it up, then enable it.
3. **Add Radarr:** Settings → Services → Add Radarr Server. Default Server on; Host **`radarr`**,
   Port `7878`, API Key (Radarr → Settings → General); then set Quality Profile 1080p, Root Folder
   `/videos/Movies`, and **Enable Automatic Search**.
4. **Add Sonarr:** Host **`sonarr`**, Port `8989`, Sonarr's API key, Root Folder `/videos/TV Shows`,
   Language Profile **English** (or the only one offered).
   - **Anime routing:** in the same Sonarr server form set **Anime Series Type = Anime**, **Anime
     Quality Profile = HD-1080p**, **Anime Root Folder = `/videos/Anime`**. Seerr then sends anime
     requests there automatically (needs Nyaa/AnimeTosho in Prowlarr — see the `prowlarr` skill).
5. **Jellyfin:** Jellyfin is the remote player (`https://jellyfin.<domain>` — see the `jellyfin` /
   `remote-access` skills). Seerr can also point at it under Settings → Services (Hostname `jellyfin`,
   Port `8096`, API key from Jellyfin → Dashboard → API Keys); Plex stays connected for at-home use.
6. **Auto-approve (optional):** Settings → Users → Default Permissions → enable **Auto-Approve** so
   requests go straight to Radarr/Sonarr without a manual approval step.

## Gotchas
- Hosts are `radarr` / `sonarr` / `host.docker.internal` — never `localhost` (that points at the
  Seerr container itself).
- The Quality Profile / Root Folder dropdowns only populate **after** the connection succeeds; if
  they stay empty, the host or API key is wrong.
- **Everything shows "Unavailable" after it downloaded?** The Plex libraries aren't enabled in
  Settings → Plex. Enable all (incl. Anime), **Sync Libraries** if one is missing, then run **Plex
  Full Scan + Availability Sync** (Settings → Jobs). A show may read **Partially Available** when some
  specials/OVAs aren't present — that's expected once the main seasons are in.
- Reachable remotely at `https://seer.<domain>` via the Caddy reverse proxy (see the
  `remote-access` skill) — no app needed.

## Check
`docker logs seerr` (look for "Server ready on port 5055") ·
`Invoke-WebRequest http://localhost:5055 -UseBasicParsing`.
