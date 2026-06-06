---
name: bazarr
description: >-
  Set up or fix Bazarr (automatic subtitles, optional) in the Media Finder stack — web UI on
  http://localhost:6767. Use to connect Bazarr to Radarr and Sonarr and add subtitle providers.
---

# Bazarr (port 6767) — subtitles (optional)

Web UI: http://localhost:6767.

## Setup
1. **Connect Sonarr:** Settings → Sonarr → Host `sonarr`, Port `8989`, Sonarr API key. Test → Save.
2. **Connect Radarr:** Settings → Radarr → Host `radarr`, Port `7878`, Radarr API key. Test → Save.
3. **Languages:** Settings → Languages → create a profile (e.g. English) and set it as the default
   for series and movies.
4. **Providers:** Settings → Providers → add one or more (e.g. OpenSubtitles.com; a free account
   helps).

## Notes
- Bazarr reads the library at `/videos`, so no extra path mapping is needed.
- API keys come from each app's Settings → General (or `C:\MediaStack\config\<app>\config.xml`).

## Check
`docker logs bazarr` · `Invoke-WebRequest http://localhost:6767 -UseBasicParsing`.
