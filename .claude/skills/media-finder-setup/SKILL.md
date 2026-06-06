---
name: media-finder-setup
description: >-
  Index and overview for the Media Finder stack (Plex + qBittorrent + Prowlarr + Radarr + Sonarr +
  Seerr + Bazarr + FlareSolverr, run with Docker Desktop on Windows). Use to get the big picture
  and the correct install order, then load each app's own skill for the detail. Start here when a
  user wants to build, run, or understand the whole stack.
---

# Media Finder — overview & install order

Media Finder is a Docker stack that automatically finds, downloads, organises, and serves movies
and TV to Plex. The user is usually **non-technical**: guide one screen at a time, give exact
values to type, verify each step, and ask for the exact error text when something breaks.
`README.md` at the project root has the full human-facing steps.

## The apps (each has its own skill)

| App | Port | Job | Skill |
|---|---|---|---|
| qBittorrent | 8080 | Downloads the torrents | `qbittorrent` |
| Prowlarr | 9696 | Manages search sites, syncs them to Radarr/Sonarr | `prowlarr` |
| Radarr | 7878 | Movies | `radarr` |
| Sonarr | 8989 | TV | `sonarr` |
| Seerr | 5055 | Request page (rebranded Overseerr) | `seerr` |
| Bazarr | 6767 | Subtitles (optional) | `bazarr` |
| FlareSolverr | 8191 | Gets past Cloudflare (background) | `flaresolverr` |
| Plex | 32400 | Plays everything (native Windows, not Docker) | `plex` |

## Install order

1. Install **Docker Desktop**; confirm it's running.
2. Open **PowerShell inside the project folder** (File Explorer → click the address bar → type
   `powershell` → Enter). Every `docker compose` / `Copy-Item` command runs from here.
3. Create folders, then `Copy-Item .env.example .env` and set `TZ` and `VIDEOS_ROOT`.
4. `docker compose up -d`.
5. Connect the apps in this order — load each app's skill for detail:
   **qbittorrent → prowlarr → radarr → sonarr → plex (libraries + notifications) → seerr →
   bazarr (optional)**. **flaresolverr** has no setup of its own; it is added inside Prowlarr.
6. Test end-to-end: request *Big Buck Bunny* (Creative-Commons) in Seerr and follow it to Plex.

## Content types

- **Movies / TV:** request in Seerr — works out of the box.
- **Anime:** add **Nyaa** + **AnimeTosho** in Prowlarr and set Seerr's anime routing (see the
  `prowlarr`, `sonarr`, and `seerr` skills), then request in Seerr.
- **K-drama:** request in Seerr — covered by the public trackers; set Bazarr subtitles.
- **Nigerian / Nollywood:** not on torrents and not self-hosted here — watch it on official
  Nollywood **YouTube** channels (free) or a streaming app (Netflix Naija / Showmax).

## True for the whole stack

- **Persistence:** containers are disposable; data lives on disk (`Videos` + `C:\MediaStack`).
  Restart, update, or rebuild loses nothing.
- **Routine stop/start:** `docker compose stop` / `docker compose start`. Use `down`/`up` only
  after editing the compose file.
- **Storage:** recommend a large HDD over an SSD; this Windows setup keeps two copies of a file
  while it seeds, so enable "Remove Completed" in Radarr/Sonarr + a qBittorrent ratio limit to
  reclaim space.
- **Legal:** the tools are legitimate; downloading copyrighted content without a licence is not.

## Check the whole stack

`docker compose ps` (status/health) · `docker logs <app>` · `docker compose config --services`.
