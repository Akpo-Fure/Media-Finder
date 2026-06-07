---
name: jellyfin
description: >-
  Set up or use Jellyfin in the Media Finder stack (web UI on http://localhost:8096) — the free,
  open-source media player used for remote watching (no paywall, plays in a browser). Reads the same
  library as Plex. Use for Jellyfin libraries, remote streaming, or connecting Seerr to Jellyfin.
---

# Jellyfin (port 8096) — free player, used for remote watching

Jellyfin plays the **same library as Plex** but with **no remote-streaming paywall** and **in a web
browser** (no app needed). That's why the stack uses it for remote viewing: Plex's remote playback
requires a paid Plex Pass and the free workarounds are blocked — so **Plex is the at-home player and
Jellyfin is the remote player**, reached at `https://jellyfin.<domain>` via the reverse proxy (see the
`remote-access` skill).

## Setup
1. Open http://localhost:8096 → choose language.
2. Create the **admin account** (username + password).
3. **Add media libraries** (Add Media Library):
   - **Movies** → folder `/media/Movies`
   - **Shows** → `/media/TV Shows`
   - **Shows** → `/media/Anime` (name it "Anime"). Set this library's **Metadata Language** to
     **English (US)** — an anime library left on another language can match the wrong entry (a
     spin-off) and show odd artwork and non-English episode titles.
   Jellyfin mounts `${VIDEOS_ROOT}` at `/media` — the **same files Plex uses**, so nothing
   re-downloads.
4. Finish the wizard.

## Notes
- Image is the official **`jellyfin/jellyfin`** (Docker Hub).
- Config: `C:\MediaStack\config\jellyfin`; transcode cache: `…\jellyfin-cache`.
- Remote access + HTTPS are handled by Caddy — see the `remote-access` skill.
- **Library stays in sync automatically:** Radarr/Sonarr trigger a Jellyfin scan on import (the
  Jellyfin notification in their settings — see the `radarr`/`sonarr` skills). A **manual import** in
  Sonarr/Radarr does not trigger it, so scan by hand afterward (Dashboard → Scheduled Tasks → Scan All
  Libraries).
- **API key** for Seerr and the Homepage dashboard: Jellyfin → Dashboard → API Keys.
- **Wrong artwork / Japanese titles on anime?** The Anime library's metadata language isn't English —
  set it to English (US), then **re-identify** the affected series to the correct entry and refresh
  metadata + images.
- Seerr can point at Jellyfin so "Available" reflects its library (see the `seerr` skill).

## Check
`docker logs jellyfin` · `Invoke-WebRequest http://localhost:8096 -UseBasicParsing` (expect 200/302).
