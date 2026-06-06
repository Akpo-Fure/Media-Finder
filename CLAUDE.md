# Media Finder — guidance for Claude Code

Media Finder is a self-hosted media stack (qBittorrent + Prowlarr + Radarr + Sonarr + Seerr + Bazarr
+ Jellyfin + Plex, plus a Caddy reverse proxy, run with Docker Desktop on Windows) that
auto-downloads movies/TV/anime and serves them at home (Plex/Jellyfin) and remotely (Jellyfin over
HTTPS).

## The person using this is non-technical

They may simply open this folder and say "help me set up my media server." When they ask for help or
seem unsure, **use the `media-finder-setup` skill** and guide them through the whole setup. For a
specific app, use that app's skill: `qbittorrent`, `prowlarr`, `radarr`, `sonarr`, `seerr`,
`bazarr`, `flaresolverr`, `plex`, `jellyfin`, `remote-access`.

**Content types:** movies, TV, K-drama, and anime are all requested in Seerr the same way. Anime
just relies on the Nyaa/AnimeTosho indexers and the Sonarr "Anime" fields, which are set up
alongside everything else during normal setup.

## How to guide them

- **Do the safe steps for them.** Run the commands yourself — create the folders, copy
  `.env.example` to `.env` and fill in the values they give you, `docker compose up -d`, read logs,
  check ports, query the apps' REST APIs. Don't make a non-technical user type terminal commands
  they don't understand.
- **One step at a time.** Give the exact value to enter on each screen, then verify it worked before
  moving on.
- **Plain language.** No jargon without a one-line explanation.
- **On errors,** ask them to paste the exact message or a screenshot, then fix it.

## Safety

- Confirm before anything destructive or outward-facing (deleting data, exposing ports to the
  internet).
- `.env` is private — do not print its secrets (API keys, VPN keys) into the chat unnecessarily.
- Don't run `docker compose down` while a download is active — it can leave qBittorrent in a
  crash-loop (see the `qbittorrent` skill). Prefer `docker compose stop` / `start`.

## Environment

Windows + PowerShell + Docker Desktop. Run `docker compose` and `Copy-Item` from this project
folder. The library lives in the user's `Videos` folder; downloads and app config live under
`C:\MediaStack`. Paths come from `.env`.

## Gotchas that bite most often (full detail in the per-app skills)

1. qBittorrent settings only save when you scroll to the bottom and click **Save**.
2. In qBittorrent, set **Bypass auth for whitelisted subnets = `172.16.0.0/12`** so Radarr/Sonarr
   connect without a password (otherwise they show "Authentication Failure").
3. The Radarr/Sonarr download-client **Host is `qbittorrent`**, never `localhost`.
4. Set the Quality Profile to **1080p** (4K is huge and slow with few seeders).
5. Remote watching is via **Jellyfin** (`jellyfin.<domain>`), not Plex — Plex's remote streaming is
   paywalled. Caddy + DDNS handle remote access; expose only Jellyfin + Seerr, never the admin apps.

## Reference

- `README.md` — the full human-facing setup guide.
- `.claude/skills/` — one skill per app, with deep detail and troubleshooting.
