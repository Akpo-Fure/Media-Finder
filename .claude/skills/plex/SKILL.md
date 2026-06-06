---
name: plex
description: >-
  Set up Plex libraries and connect the Media Finder stack to Plex. Plex runs natively on Windows
  (not in Docker) on port 32400. Use to add the Movies/TV libraries, point Radarr/Sonarr/Seerr at
  Plex, or fix Plex not seeing new media or your drive.
---

# Plex (port 32400) — the player (native Windows)

Plex is the user's existing native Windows app — **not** a container. It's the **at-home** player.
For **remote** watching the stack uses **Jellyfin** instead (Plex's remote streaming is paywalled
and the free workarounds are blocked) — see the `jellyfin` and `remote-access` skills. Both read the
same library files.

## Setup (libraries)
1. Settings (wrench) → Manage → Libraries → Add Library (or the + in the sidebar).
2. **Movies** → folder `C:\Users\<you>\Videos\Movies`.
3. **TV Shows** → folder `C:\Users\<you>\Videos\TV Shows`.
   - **Anime** (TV Shows type) → `C:\Users\<you>\Videos\Anime`. (Advanced: the HAMA agent matches
     anime better, but the default agent works.)
4. Settings → Library → tick **Scan my library automatically** (a backup to the Radarr/Sonarr
   notifications).

## How the stack connects to Plex
- Radarr/Sonarr (Settings → Connect → Plex Media Server) and Seerr point at Plex using host
  **`host.docker.internal`**, port **`32400`** — because they run in containers and Plex is on the
  host. Fallback: the PC's LAN IPv4 from `ipconfig`.

## Gotchas
- **Plex can't see your `C:` drive** when adding a library → Plex is running as a different Windows
  user; run it as your own account (or grant that account read access to the Videos folder).
- Plex uses **Windows paths** (`C:\Users\...\Videos\...`), not the container paths (`/videos/...`).
- New media not appearing → confirm the Radarr/Sonarr Plex "Connect" is set up and the Plex library
  folder matches the Videos path.

## Check
In the Plex web app the Movies/TV libraries should scan and play. From a container, test
reachability with:
`docker exec radarr curl -s -o /dev/null -w "%{http_code}" http://host.docker.internal:32400/identity`
