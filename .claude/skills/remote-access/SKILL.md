---
name: remote-access
description: >-
  Set up or fix remote access to the Media Finder stack from outside the home — public HTTPS links
  with no client app, via a Caddy reverse proxy + router port-forward, Let's Encrypt certificates,
  and DDNS for the dynamic IP. Exposes Jellyfin (watch), Seerr (request), and optionally Plex.
---

# Remote access — Caddy reverse proxy + port-forward + HTTPS + DDNS

Lets the user watch and request from anywhere via public links, **no app on the viewer side**, all
in Docker. (Plex's remote streaming is paywalled, so **Jellyfin is the remote player**.)

## What's exposed — and what is NOT
- Public: **`jellyfin.<domain>`** (watch) and **`seer.<domain>`** (Seerr), HTTPS, free.
  **`plex.<domain>`** can also be published to reach Plex's web UI remotely — but Plex's remote
  *streaming* needs a paid Plex Pass, so use Jellyfin to actually watch; exposing Plex is optional.
- **Never expose** Radarr / Sonarr / qBittorrent / Prowlarr — they stay LAN-only (not in the Caddyfile).

## Pieces (both in docker-compose.yml)
- **caddy** — reverse proxy with **automatic Let's Encrypt HTTPS**. Publishes ports **80 + 443**.
  Routes live in `C:\MediaStack\config\caddy\Caddyfile` — one block per subdomain, e.g.
  `jellyfin.<domain> { reverse_proxy jellyfin:8096 }` and `seer.<domain> { reverse_proxy seerr:5055 }`;
  optionally `plex.<domain> { reverse_proxy host.docker.internal:32400 }` (Plex is native Windows).
- **ddns-updater** — keeps the domain's A records pointed at the dynamic public IP. Config (with the
  registrar's DDNS password) is `C:\MediaStack\config\ddns-updater\config.json` — outside the repo,
  never committed.

## Setup
1. **DNS:** at your registrar / DNS provider (must support Dynamic DNS — see ddns-updater's
   supported providers) add A records `jellyfin` + `seer` (and `plex` if you expose it) → your
   public IP. DDNS keeps them current as the IP changes.
2. **Check you're NOT behind CGNAT first:** the router's WAN IP must equal your public IP
   (`Invoke-RestMethod https://api.ipify.org`). If they differ (CGNAT), port-forward can't work — use
   a tunnel instead.
3. **Router:** forward TCP **80 + 443** to the PC. (Wording varies by router — look for **Port
   forwarding** and a **Reservation / static lease** in your router's app or admin page at
   `http://192.168.x.x`; reserve the PC's IP too.)
4. **Start Caddy** (`docker compose up -d caddy`) — it auto-fetches the certs, which also proves
   port 80 is open.
5. **Windows Firewall:** allow inbound 80 + 443.
6. Test from a phone on mobile data: `https://jellyfin.<domain>` plays; `https://seer.<domain>`
   loads Seerr.

## Gotchas / fixes
- After editing the **Caddyfile**, reload Caddy:
  `docker exec caddy caddy reload --config /etc/caddy/Caddyfile`.
- After editing **ddns-updater** `config.json`, restart it (`docker restart ddns-updater`) — it only
  reads its config at startup.
- Cert won't issue → DNS isn't pointing at your IP yet, or port 80 isn't forwarded.
- **Plex remote streaming needs a paid Plex Pass** (the reverse-proxy / "LAN Networks" trick is
  blocked, 2025/26) — the `plex.<domain>` route reaches Plex's web UI, but to actually watch remotely
  use **Jellyfin**.

## Check
`docker logs caddy` (look for "certificate obtained successfully"); from the PC, confirm routing:
`curl.exe -s -o NUL -w "%{http_code}" --resolve jellyfin.<domain>:443:127.0.0.1 https://jellyfin.<domain>`.
