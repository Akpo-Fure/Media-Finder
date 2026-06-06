---
name: qbittorrent
description: >-
  Set up or fix qBittorrent (the torrent downloader) in the Media Finder stack — web UI on
  http://localhost:8080. Use for the qBittorrent login, save paths, the subnet whitelist that lets
  Radarr/Sonarr connect, seeding limits, or a qBittorrent web UI that won't load.
---

# qBittorrent (port 8080) — the downloader

Web UI: http://localhost:8080. Guide a non-technical user one step at a time; verify each step.

## First login
- Get the temporary password: `docker logs qbittorrent` (look for "temporary password").
- Log in as `admin` + that password. It **changes on every restart** until a real one is set, so
  do the next step now.

## Setup (gear/Settings icon)
1. **Web UI tab → Authentication:** set your own Username + Password.
2. **Web UI tab → Authentication:** tick **"Bypass authentication for clients in whitelisted IP
   subnets"** and enter **`172.16.0.0/12`**. This is what lets Radarr/Sonarr connect without a
   password. (Find the exact subnet with `docker network inspect mediafinder_default`.)
3. **Downloads tab:** Default Save Path `/data/torrents`; Default Torrent Management Mode
   **Automatic**.
4. **Scroll to the very bottom and click Save.** qBittorrent saves nothing until you click it —
   the #1 beginner trap.
5. **BitTorrent tab:** Share Ratio Limit `2.0`, action stop/pause (so finished files don't fill
   the disk).

Do **not** create `movies`/`tv` categories by hand — Radarr and Sonarr create them automatically.

## How other apps reach it
Radarr/Sonarr download client → **Host `qbittorrent`, Port `8080`** (never `localhost`).

## Gotchas / fixes
- Radarr/Sonarr **"Authentication Failure"** → the whitelist (step 2) is missing/wrong.
- Settings won't stick → you didn't press **Save** (step 4).
- **Web UI won't load after a restart** (stale lock from being stopped mid-download):
  `docker exec qbittorrent rm -f /config/qBittorrent/ipc-socket /config/qBittorrent/lockfile`,
  then wait ~15 s. Prefer `docker compose stop`/`start` over `down`/`up`.

## Check
`docker logs qbittorrent` · `Invoke-WebRequest http://localhost:8080 -UseBasicParsing`.
