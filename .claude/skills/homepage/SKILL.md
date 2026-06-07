---
name: homepage
description: >-
  Set up or edit the Homepage dashboard in the Media Finder stack (http://localhost:9999) — one page
  with tiles, live widgets, and quick links for every app. Use to add/adjust tiles or widgets, fix
  "host validation failed", or wire an app's API key into a widget.
---

# Homepage (port 9999) — the dashboard

One page at `http://localhost:9999` showing all the apps with live stats + quick links.
**Local-only** (not exposed via Caddy) because it holds API keys. Host port **9999** → container 3000.

## Config (in `C:\MediaStack\config\homepage` — outside the repo, keys not committed)
- `settings.yaml` — title, theme, group layout.
- `services.yaml` — the tiles, grouped (Media / Management / Downloads). Each has an `href` (the
  clickable `localhost` link) and an optional `widget` (live stats) that uses the app's API key and
  its **container** URL (e.g. `url: http://radarr:7878` — Homepage is on the same Docker network).
- `widgets.yaml` — info widgets (CPU/memory, search bar).
- Homepage also auto-creates `docker.yaml` / `bookmarks.yaml` / `custom.css` etc. (unused defaults).

## Widgets
Radarr, Sonarr, Prowlarr, Seerr (`overseerr` type), qBittorrent, and Jellyfin have **live widgets**;
each reads its app over the Docker network with the app's own API key. Bazarr and Plex are quick-link
tiles — add their widgets the same way if you want live stats:
- qBittorrent → `type: qbittorrent`, `url: http://qbittorrent:8080` — **no username/password** (the
  qBittorrent subnet whitelist lets Homepage in over the Docker network).
- Jellyfin → `type: jellyfin`, `url: http://jellyfin:8096`, key from Jellyfin → Dashboard → API Keys.
- Bazarr → `type: bazarr`, key from Bazarr → Settings → General.
- Plex → `type: plex`, a Plex token.

## Gotchas
- **`HOMEPAGE_ALLOWED_HOSTS`** (in `docker-compose.yml`) lists every `host:port` you open Homepage
  with; it ships as `localhost:9999`. To reach it from another device on your home network, add your
  PC's LAN IP — e.g. `HOMEPAGE_ALLOWED_HOSTS=localhost:9999,192.168.x.x:9999` — then
  `docker compose up -d homepage`. A missing/wrong entry shows "host validation failed".
- A widget's `url` is the **container** name/port (`radarr:7878`), not `localhost`; the `href` is the
  clickable `localhost` link.
- Keep it **local** — don't expose it via Caddy (it holds API keys). If you ever must, put auth in front.

## Check
`docker logs homepage` · `Invoke-WebRequest http://localhost:9999 -UseBasicParsing` (expect 200).
