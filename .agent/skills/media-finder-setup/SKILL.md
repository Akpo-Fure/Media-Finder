---
name: media-finder-setup
description: >-
  Index and overview for the Media Finder stack (qBittorrent + Prowlarr + Radarr + Sonarr + Seerr +
  Bazarr + FlareSolverr + Jellyfin + Plex, with a Caddy reverse proxy for remote access, run with
  Docker Desktop on Windows). Use to get the big picture and install order, then load each app's own
  skill for detail. Start here when a user wants to build, run, or understand the whole stack.
---

# Media Finder — overview & install order

Media Finder is a Docker stack that automatically finds, downloads, organises, and serves movies,
TV, anime, and K-drama. You **request** titles in Seerr; they download and land in your library; you
**watch** in Plex (at home) or **Jellyfin** (anywhere, free). It's reachable from outside the home
over HTTPS with **no app on the viewer side**. The user is usually **non-technical**: guide one
screen at a time, give exact values, verify each step, and ask for the exact error text when
something breaks. `README.md` has the full human-facing steps.

## The apps (each has its own skill)

| App | Port | Job | Skill |
|---|---|---|---|
| qBittorrent | 8080 | Downloads the torrents | `qbittorrent` |
| Prowlarr | 9696 | Manages search sites, syncs them to Radarr/Sonarr | `prowlarr` |
| Radarr | 7878 | Movies | `radarr` |
| Sonarr | 8989 | TV (and anime) | `sonarr` |
| Seerr | 5055 | Request page (works with Plex + Jellyfin) | `seerr` |
| Bazarr | 6767 | Subtitles (optional) | `bazarr` |
| FlareSolverr | 8191 | Gets past Cloudflare (background) | `flaresolverr` |
| Jellyfin | 8096 | Free playback incl. **remote** watching (browser) | `jellyfin` |
| Plex | 32400 | Local playback at home (native Windows) | `plex` |

Plus background infra for remote access — **Caddy** (reverse proxy + HTTPS) and **ddns-updater**
(keeps DNS current): see the `remote-access` skill.

## Install order

1. Install **Docker Desktop**; confirm it's running.
2. Open **PowerShell inside the project folder** (File Explorer → address bar → type `powershell`).
3. Create folders, then `Copy-Item .env.example .env` and set `TZ` and `VIDEOS_ROOT`.
4. `docker compose up -d`.
5. Connect the apps in this order — load each app's skill for detail:
   **qbittorrent → prowlarr → radarr → sonarr → plex & jellyfin (libraries) → seerr →
   bazarr (optional)**. **flaresolverr** has no setup of its own (added inside Prowlarr).
6. **Remote access:** set it up via the `remote-access` skill (domain DNS + Caddy + router
   port-forward + DDNS) so Jellyfin and Seerr are reachable from anywhere.
7. Test end-to-end: request *Big Buck Bunny* (Creative-Commons) in Seerr → it appears in Jellyfin/Plex.

## Content types

**Movies, TV, K-drama, and anime are all requested in Seerr — the same way.** Set up together in the
normal flow:
- The Prowlarr indexer list includes **Nyaa + AnimeTosho** (anime) alongside the general trackers.
- Sonarr gets `/videos/TV Shows` **and** `/videos/Anime` root folders; Seerr's Sonarr server has the
  **Anime** fields set so anime sorts correctly.
- K-drama needs nothing special (general trackers cover it; set Bazarr subtitles).

## Watching: Plex vs Jellyfin
- **At home:** Plex or Jellyfin — both free.
- **Remotely:** **Jellyfin** (`https://jellyfin.<domain>`, free, in a browser). Plex's remote
  streaming is paywalled (Plex Pass) and the free workarounds are blocked, so Jellyfin is the remote
  player. Both read the **same** library files.

## True for the whole stack

- **Persistence:** containers are disposable; data lives on disk (`Videos` + `C:\MediaStack`).
- **Routine stop/start:** `docker compose stop` / `start`. Use `down`/`up` only after editing compose.
- **Storage:** big HDD over SSD; this Windows setup keeps two copies while seeding — enable "Remove
  Completed" in Radarr/Sonarr + a qBittorrent ratio limit to reclaim space.
- **Remote access:** Jellyfin (watch) + Seerr (request) are public over HTTPS via Caddy +
  port-forward + DDNS; admin apps stay LAN-only — see the `remote-access` skill.
- **Legal:** the tools are legitimate; downloading copyrighted content without a licence is not.

## Check the whole stack
`docker compose ps` (status/health) · `docker logs <app>` · `docker compose config --services`.
