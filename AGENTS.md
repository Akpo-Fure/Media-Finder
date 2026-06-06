# Media Finder — guidance for AI agents

Media Finder is a self-hosted media stack (Plex + qBittorrent + Prowlarr + Radarr + Sonarr + Seerr
+ Bazarr, run with Docker Desktop on Windows). The person using it is typically **non-technical**.

If you are an agent helping with this project, load the **`media-finder-setup`** skill at
`.agent/skills/media-finder-setup/SKILL.md` for the overview and install order, then the matching
per-app skill at `.agent/skills/<app>/SKILL.md` (`qbittorrent`, `prowlarr`, `radarr`, `sonarr`,
`seerr`, `bazarr`, `flaresolverr`, `plex`) for detail.

Guide one step at a time, run the safe commands for the user, verify each step, and never print
`.env` secrets. `README.md` is the full human-facing guide.

(Claude Code users: see `CLAUDE.md` and `.claude/skills/` — same content.)
