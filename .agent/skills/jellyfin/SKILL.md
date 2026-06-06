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
now requires a paid Plex Pass and the free workarounds are blocked — so **Plex is the at-home player
and Jellyfin is the remote player**, reached at `https://jellyfin.<domain>` via the reverse proxy
(see the `remote-access` skill).

## Setup
1. Open http://localhost:8096 → choose language.
2. Create the **admin account** (username + password).
3. **Add media libraries** (Add Media Library):
   - **Movies** → folder `/media/Movies`
   - **Shows** → `/media/TV Shows`
   - **Shows** → `/media/Anime` (name it "Anime")
   Jellyfin mounts `${VIDEOS_ROOT}` at `/media` — the **same files Plex uses**, so nothing
   re-downloads.
4. Finish the wizard; metadata defaults are fine.

## Notes
- Image is the **official `jellyfin/jellyfin`** (Docker Hub) — used instead of the LinuxServer image,
  which had a flaky layer pull from `lscr.io`.
- Config: `C:\MediaStack\config\jellyfin`; transcode cache: `…\jellyfin-cache`.
- Remote access + HTTPS are handled by Caddy — see the `remote-access` skill.
- **Connect Seerr to Jellyfin** (Seerr supports it) so "Available" status reflects the Jellyfin
  library.

## Check
`docker logs jellyfin` · `Invoke-WebRequest http://localhost:8096 -UseBasicParsing` (expect 200/302).
