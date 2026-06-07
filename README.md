# Media Finder

Media Finder is a set of free apps that work together to find movies and TV shows,
download them automatically, organise them into folders, and add them to your Plex
server so you can watch them. You ask for a movie on a simple web page, and a while
later it appears in Plex.

This guide is written for a complete beginner on a Windows PC. Follow it from top to
bottom, in order. You do not need any technical experience.

> **Easiest path — let Claude Code do it.** If you have Claude Code, open this folder in it and say
> *"help me set up Media Finder"*. It will run the steps for you and guide you the whole way.
> Prefer to do it by hand? Just follow the steps below.

---

## What you need before you start

1. A **Windows 10 or 11 PC** that you can leave switched on when you want to use the server.
2. **Plex** already installed and signed in on that PC. (Get it from https://www.plex.tv if you
   don't have it.)
3. **Free hard-drive space** for your movies and shows. A large normal hard drive (HDD) is best —
   see "Saving disk space" near the end.
4. About **30–45 minutes** for the one-time setup.

### A few words you will see, explained simply

- **Docker Desktop** – a program that runs each app in its own neat box. You install it once.
- **Container** – one of those boxes (one per app). You never touch the inside; you just start
  and stop them.
- **PowerShell** – the black command window in Windows where you paste a line and press Enter.
- **Web UI** – the settings page for an app that you open in your web browser.

---

## The apps and their web addresses

After setup, each app has its own page you open in a browser:

| App | Address | What it does |
|---|---|---|
| qBittorrent | http://localhost:8080 | Downloads the files |
| Prowlarr | http://localhost:9696 | Manages the websites it searches |
| Radarr | http://localhost:7878 | Handles movies |
| Sonarr | http://localhost:8989 | Handles TV shows |
| Seerr | http://localhost:5055 | The page where you request things |
| Bazarr | http://localhost:6767 | Gets subtitles (optional) |
| FlareSolverr | (works in the background) | Helps reach protected search sites |
| Jellyfin | http://localhost:8096 | Free player — and how you watch from anywhere |
| Plex | your existing Plex | Plays everything at home |
| Homepage | http://localhost:9999 | Dashboard — all apps, stats & links in one page |

You do not need to sign up for any accounts except the Plex you already have.

---

# Part 1 – Install and start

> **Before you start:** the project files — `docker-compose.yml`, `.env.example`, and this
> `README.md` — must all sit together in one folder (for example `Documents\Media Finder`). That
> single folder is what this guide means by "this folder".

## Step 1. Install Docker Desktop

1. Download Docker Desktop from https://www.docker.com/products/docker-desktop/
2. Run the installer, accept the defaults, and restart the PC if it asks. **If Windows offers to
   install WSL2, allow it** — Docker needs it.
3. Open **Docker Desktop** and wait until it says it is running (a whale icon appears near the
   clock). Leave it running whenever you want the server available.

## Step 2. Open PowerShell in this folder

Every command in this guide must be run from inside the project folder (the one holding
`docker-compose.yml`). The easy way to open PowerShell there:

1. Open the project folder in **File Explorer**.
2. Click the **address bar** at the top so the folder path is highlighted.
3. Type `powershell` and press **Enter**.

A PowerShell window opens, already pointing at this folder. Keep it open and use it for every
command below.

## Step 3. Create the folders

This makes the folders for downloads, app settings, and your Plex library. Paste this whole block
into PowerShell and press Enter. It uses your own Windows account automatically, so there is
nothing to change:

```powershell
$dirs = @(
  'C:\MediaStack\data\torrents\movies','C:\MediaStack\data\torrents\tv',
  'C:\MediaStack\config\qbittorrent','C:\MediaStack\config\prowlarr',
  'C:\MediaStack\config\radarr','C:\MediaStack\config\sonarr',
  'C:\MediaStack\config\flaresolverr','C:\MediaStack\config\seerr',
  'C:\MediaStack\config\bazarr','C:\MediaStack\config\gluetun',
  'C:\MediaStack\config\jellyfin','C:\MediaStack\config\jellyfin-cache',
  'C:\MediaStack\config\caddy','C:\MediaStack\config\ddns-updater',
  'C:\MediaStack\config\homepage',
  "$env:USERPROFILE\Videos\Movies","$env:USERPROFILE\Videos\TV Shows",
  "$env:USERPROFILE\Videos\Anime"
)
$dirs | ForEach-Object { New-Item -ItemType Directory -Force -Path $_ | Out-Null }
```

This puts your library in your own Videos folder (a `Movies` and a `TV Shows` folder inside it),
and everything else under `C:\MediaStack`.

## Step 4. Set up the settings file

In this folder there is a file called `.env.example`. Make your own copy of it called `.env`:

```powershell
Copy-Item .env.example .env
```

Open `.env` in Notepad and set these. **These two are the only things you must personalise:**

- **`TZ`** – your timezone, for example `America/New_York` or `Europe/London`.
  (Full list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
- **`VIDEOS_ROOT`** – your Videos folder. To find the exact path, run this in PowerShell:
  `echo $env:USERPROFILE`. Take what it prints (for example `C:\Users\sam`), add `/Videos` on the
  end, and write it with **forward slashes** — for example `VIDEOS_ROOT=C:/Users/sam/Videos`.

Leave the other lines as they are. Leave all the `VPN_` lines blank for now. Save and close.

## Step 5. Start everything

In the **PowerShell** window from Step 2 (it is already in this folder), run:

```powershell
docker compose up -d
```

The first time, it downloads the apps (a few minutes). When it finishes, check they are running:

```powershell
docker compose ps
```

Open each address from the table above in your browser to confirm the pages load.

> **Note – how to stop and start later:** to pause everything use `docker compose stop`, and to
> turn it back on use `docker compose start`. Only use `docker compose down` if you have edited the
> setup files. Your downloads and settings are always kept either way.

---

# Part 2 – Connect the apps

Do these in the order shown. It is mostly typing values and clicking Save. The lines marked
**Important** are the ones beginners most often get wrong.

## A. qBittorrent (the downloader) – http://localhost:8080

**Log in for the first time.** Go to PowerShell and run:

```powershell
docker logs qbittorrent
```

Look for a line that says *temporary password*. Log in with username `admin` and that password.

> **Important:** that temporary password changes every time the app restarts, so set your own
> password now (next step).

1. Click the **gear/Settings** icon, then the **Web UI** tab.
2. Under **Authentication**, set a **Username** and **Password** you will remember.
3. **Still under Authentication**, tick **"Bypass authentication for clients in whitelisted IP
   subnets"** and type this in the box: `172.16.0.0/12`
   - **Important:** this lets the other apps talk to qBittorrent without a password. If you skip it,
     Radarr and Sonarr will later show "Authentication Failure".
4. Click the **Downloads** tab and set:
   - **Default Save Path**: `/data/torrents`
   - **Default Torrent Management Mode**: **Automatic**
5. **Important:** scroll to the very bottom of the settings and click **Save**. qBittorrent does
   not save anything until you click this button.
6. Open the **BitTorrent** settings tab, turn on a **Share Ratio Limit**, set it to **2.0**, choose
   **stop/pause** when reached, and Save. (This stops finished files seeding forever and filling
   your disk.)

> **Note:** you do not need to create "movies" or "tv" categories in qBittorrent yourself. Radarr
> and Sonarr create them automatically the first time they send a download.

## B. Prowlarr (the search manager) – http://localhost:9696

When it first opens it asks you to make a username and password. Do that, then:

1. **Add the helper for protected sites.** Go to **Settings → Indexers**, scroll to **Indexer
   Proxies**, click the **+**, choose **FlareSolverr**:
   - **Host**: `http://flaresolverr:8191`
   - **Tags**: type `flaresolverr` and press Enter
   - Click **Test**, then **Save**.
2. **Add search sites.** Go to the **Indexers** page, click **Add Indexer**, and add these (they
   cover movies, TV, K-drama, and anime): **Knaben**, **The Pirate Bay**, **1337x**, **YTS** (movies
   only), **Nyaa**, and **AnimeTosho** (the last two are the anime trackers). Add each and **Test**.
   - **Tags:** leave the Tags box **blank** for normal sites (Knaben, 1337x, YTS, Nyaa, etc.). Only
     if a site shows a **Cloudflare** warning, open it and add the **`flaresolverr`** tag, then Test
     again.
   - **Note:** some old sites are dead and will just error — for example **TorrentGalaxy** (closed
     down) and **EZTV** (often shows "451"). Skip those and use the ones above.
   - **K-drama** works on these trackers — no special one needed.
3. **Link Radarr and Sonarr** (come back and do this part *after* you finish sections C and D):
   - Go to **Settings → Apps**, click **+**, choose **Radarr**.
   - **Prowlarr Server**: `http://prowlarr:9696`
   - **Radarr Server**: `http://radarr:7878`
   - **API Key**: open Radarr → **Settings → General** and copy its API Key into this box.
   - **Sync Level**: **Full Sync**. Test, then Save.
   - Do the same again choosing **Sonarr** (`http://sonarr:8989`, and Sonarr's own API Key).
   - From now on, every search site you add appears automatically in Radarr and Sonarr.

## C. Radarr (movies) – http://localhost:7878

Set a username and password when asked, then:

1. **Settings → Media Management**, scroll to **Root Folders**, click **+**, and add:
   `/videos/Movies`
2. **Settings → Download Clients**, click **+**, choose **qBittorrent**:
   - **Host**: `qbittorrent`  — **Important:** type the word `qbittorrent`, not `localhost`.
   - **Port**: `8080`
   - **Category**: `movies`
   - Click **Test** (it should turn green), then **Save**.
3. **Avoid huge, slow downloads.** Go to **Settings → Profiles → Quality Profiles** and use the
   **HD-1080p** profile (or edit your profile so 1080p is the top quality and 4K is turned off).
   4K files are enormous and can take days; 1080p is small and fast.

## D. Sonarr (TV shows) – http://localhost:8989

Exactly like Radarr, with TV values:

1. **Root Folders**: add `/videos/TV Shows` **and** `/videos/Anime` (anime gets its own folder — it
   uses absolute episode numbering).
2. **Download Client → qBittorrent**: Host `qbittorrent`, Port `8080`, Category `tv`. Test, Save.
3. Set the **Quality Profile** to **1080p** as well.

## E. Tell Plex and Jellyfin to update automatically

So new downloads show up in your players on their own, set this in **both Radarr and Sonarr**:

1. Go to **Settings → Connect**, click **+**, choose **Plex Media Server**.
2. **Hostname**: `host.docker.internal`  •  **Port**: `32400`
3. Click **Authenticate with Plex.tv** and sign in to approve it.
4. Leave the triggers (On Import, On Upgrade) ticked. Click **Test**, then **Save**.

> **Note:** if the Test fails, open PowerShell, run `ipconfig`, find your **IPv4 Address** (looks
> like `192.168.x.x`), and use that number instead of `host.docker.internal`.

If you also use **Jellyfin** (the remote player — see "Watch & request from anywhere" below), add a
second connection here the same way: **Emby / Jellyfin**, Host `jellyfin`, Port `8096`, **API Key**
from Jellyfin → Dashboard → API Keys, tick **Update Library**. Then both players update on their own.

## F. Add your libraries to Plex

In Plex (your existing app):

> **Note:** `YOURNAME` below is your Windows account folder under `C:\Users`. Not sure what it is?
> Run `echo $env:USERPROFILE` in PowerShell — the last part is your name.

1. Click **Settings (wrench) → Manage → Libraries → Add Library** (or the **+** in the sidebar).
2. Choose **Movies**, name it, and point it at `C:\Users\YOURNAME\Videos\Movies`.
3. Add another library, choose **TV Shows**, point it at `C:\Users\YOURNAME\Videos\TV Shows`.
   - **Anime** (TV Shows type) → `C:\Users\YOURNAME\Videos\Anime`.
4. In **Settings → Library**, tick **Scan my library automatically** as a backup.

## G. Seerr (the request page) – http://localhost:5055

This is the friendly page where you ask for movies and shows.

1. **Sign in with Plex.**
2. **Connect your Plex server.** On the Plex settings step, either press the button next to the
   **Server** dropdown to fetch your server automatically, or fill it in by hand:
   - **Hostname or IP Address**: `host.docker.internal` (or your `192.168.x.x` address)
   - **Port**: `32400`
   - Save, then click **Sync Libraries** and turn on **all** of them — **Movies, TV Shows, and
     Anime**. Seerr only shows a title as "Available" for libraries that are turned on here. (If a
     library is missing from the list — e.g. Anime — click **Sync Libraries** again to pick it up.)
3. **Add Radarr.** Go to **Settings → Services → Add Radarr Server**:
   - **Default Server**: on  •  **4K Server**: off
   - **Hostname or IP Address**: `radarr`  — **Important:** the word `radarr`, not localhost.
   - **Port**: `7878`
   - **API Key**: from Radarr → Settings → General
   - After it connects, set **Quality Profile** to 1080p, **Root Folder** to `/videos/Movies`,
     turn on **Enable Automatic Search**, and Save.
4. **Add Sonarr.** Same steps with **Hostname** `sonarr`, **Port** `8989`, Sonarr's API Key,
   **Root Folder** `/videos/TV Shows`, pick the available **Language Profile**, and Save.
   - In the **same form**, also set the **Anime** fields so anime sorts correctly: **Anime Series
     Type** = `Anime`, **Anime Quality Profile** = `HD-1080p`, **Anime Root Folder** = `/videos/Anime`.

## H. Bazarr (subtitles) – optional – http://localhost:6767

Connect **Sonarr** (`http://sonarr:8989`) and **Radarr** (`http://radarr:7878`) using their API
keys, set a language, and add subtitle providers under **Settings → Providers**.

---

## Test that it all works

Use a free, legal film so you are not downloading anything copyrighted:
**Big Buck Bunny** (a Creative-Commons movie that is in the movie database).

1. Open **Seerr** (http://localhost:5055), search **Big Buck Bunny**, and click **Request**.
2. It flows through on its own: Radarr searches, qBittorrent downloads, the file is moved into
   `C:\Users\YOURNAME\Videos\Movies`, and Plex shows it.
3. In Radarr you can watch progress under **Activity**.

---

## Everyday use

- Ask for **movies, TV, K-drama, and anime** in **Seerr** — all the same way.
- Sonarr automatically grabs new episodes of shows you follow as they air.
- Everything appears in **Plex and Jellyfin** by itself.
- Your **dashboard** (all apps in one page) is at `http://localhost:9999`.
- To pause or resume the whole server:
  ```powershell
  docker compose stop     # pause
  docker compose start    # resume
  ```

---

## Watch & request from anywhere (remote access)

You can use your server away from home — in any phone or laptop **browser, with no app to install** —
over your own domain with HTTPS. (Plex's remote streaming now costs money, so **Jellyfin** is the
remote player: it's free and plays in a browser.)

How it's wired (all in Docker):
- **Jellyfin** (`http://localhost:8096`) — a free player that reads the **same** Movies/TV/Anime as
  Plex. Do its quick first-run: create an admin account and add libraries pointing at `/media/Movies`,
  `/media/TV Shows`, and `/media/Anime`. For the **Anime** library, set its **Metadata Language** to
  **English (US)** so anime match the right entry and show English titles.
- **Caddy** — a reverse proxy that gives tidy HTTPS links. It exposes **Jellyfin** and **Seerr** (and
  optionally **Plex**):
  - `https://jellyfin.<your-domain>` — watch
  - `https://seer.<your-domain>` — request (Seerr)
  - `https://plex.<your-domain>` — optional; reaches Plex's web page remotely, but Plex's remote
    *streaming* needs paid Plex Pass, so use Jellyfin to actually watch.
- **ddns-updater** — keeps your domain pointed at your home IP when it changes.

To set it up:
1. Put your domain's DNS on your registrar / DNS provider (one that supports Dynamic DNS — see
   ddns-updater's provider list); add A records `jellyfin`, `seer` (and `plex` if you expose it) →
   your public IP.
2. Make sure you have a real public IP (not CGNAT): your router's WAN IP must match what
   `https://api.ipify.org` shows. If it doesn't, port-forwarding can't work.
3. Forward router ports **80 and 443** to this PC, and allow them in Windows Firewall.
4. Put your subdomains in `C:\MediaStack\config\caddy\Caddyfile`, then `docker compose up -d caddy` —
   it fetches the HTTPS certificates automatically.
5. Add your registrar's Dynamic-DNS password to `C:\MediaStack\config\ddns-updater\config.json`.

> **Only Jellyfin, Seerr (and optionally Plex) are exposed** — never Radarr/Sonarr/qBittorrent/
> Prowlarr. Use strong passwords. (Full detail is in the `remote-access` skill.)

---

## Saving disk space

- **Buy a big normal hard drive (HDD), not an SSD.** Movies are huge files that play fine from a
  cheap HDD, and HDDs cost far less per terabyte. An 8–18 TB external USB drive is the easy choice.
  Keep your SSD for Windows and the apps.
- **This setup keeps two copies for a while** — the download copy (still sharing) and the copy in
  your Videos library. To clear the extra copy automatically once a download has shared enough:
  - In **Radarr and Sonarr → Settings → Download Clients**, open the qBittorrent entry and turn on
    **Remove Completed**. Combined with the ratio limit you set in step A6, the download copy is
    deleted after it finishes sharing, leaving just the library copy.
- **Prefer 1080p over 4K** — about one-eighth the size.
- **To move your library to a bigger drive later:** change `VIDEOS_ROOT` (and/or `DATA_ROOT`) in
  the `.env` file, move the folder, then run `docker compose up -d` again.

---

## Does it keep my stuff if I restart?

Yes. The apps live in disposable containers, but all your data is saved on your real disk:

| Thing | Saved in | Kept after restart/update |
|---|---|---|
| Movies and shows | `C:\Users\YOURNAME\Videos` | Yes |
| All app settings | `C:\MediaStack\config` | Yes |
| Downloads in progress | `C:\MediaStack\data\torrents` | Yes |

You can restart Windows, update the apps, or rebuild a container, and nothing is lost.

---

## Turning on the VPN later (optional, for privacy)

Without a VPN, other people sharing a file can see your home internet address, and your internet
provider can see download activity. A VPN hides this. The VPN is already built in but switched off.
To turn it on:

1. Sign up for a VPN that allows file sharing (such as Mullvad, ProtonVPN, or IVPN), create a
   **WireGuard** configuration, and put the values into the `.env` file (`VPN_SERVICE_PROVIDER`,
   `WIREGUARD_PRIVATE_KEY`, `WIREGUARD_ADDRESSES`).
2. In `docker-compose.yml`, under `qbittorrent`, follow the comment instructions: comment out the
   `ports` block and uncomment the `network_mode` and `depends_on` lines.
3. In Radarr and Sonarr, change the qBittorrent **Host** from `qbittorrent` to `gluetun`.
4. Start with: `docker compose --profile vpn up -d`
5. Check it is working — this should show the VPN's location, not yours:
   ```powershell
   docker exec gluetun wget -qO- https://ipinfo.io
   ```

---

## Is this legal?

These apps are legal tools with many legal uses: your own ripped discs, public-domain films,
Creative-Commons releases, Linux downloads, and so on. **Downloading or sharing copyrighted films
and shows without permission is against the law in most countries.** A VPN gives privacy but does
not make it legal. You are responsible for following the law where you live. Please get content
legally.

---

## Fixes for common problems

**Radarr or Sonarr says "Unable to connect to qBittorrent".**
The Host is wrong. It must be the word `qbittorrent` (not `localhost`), with Port `8080`.

**Radarr or Sonarr says "Authentication Failure".**
Do step A3: in qBittorrent, turn on **Bypass authentication for clients in whitelisted IP subnets**
and enter `172.16.0.0/12`, then Save. After that the password no longer matters.

**A setting in qBittorrent won't stick / your password didn't save.**
You didn't press **Save**. Scroll to the very bottom of the qBittorrent settings page and click it.

**qBittorrent's page (http://localhost:8080) won't load after a restart.**
It can get stuck after being stopped mid-download. Fix it with this one line, then wait ~15 seconds:
```powershell
docker exec qbittorrent rm -f /config/qBittorrent/ipc-socket /config/qBittorrent/lockfile
```
To avoid it, pause with `docker compose stop` (not `down`).

**A search site won't add (shows "451" or keeps redirecting).**
That site is blocked or closed. Use the working ones: Knaben, The Pirate Bay, 1337x, YTS. For
Cloudflare warnings, add the `flaresolverr` tag. If almost everything fails, your internet provider
is blocking these sites — turn on the VPN.

**Downloads are extremely slow / it says it will take days.**
It picked a 4K file. Set the Quality Profile to **1080p** in Radarr and Sonarr, or use **Interactive
Search** and choose a 1080p result with lots of seeders.

**An anime downloaded but won't appear ("Episode X was not found in the grabbed release").**
Anime "complete series" packs sometimes use episode numbering Sonarr can't match. In **Sonarr →
Activity** (or **Wanted**), use **Manual Import**: point it at the download folder and Sonarr re-reads
each file and files it correctly. After a manual import, scan your Plex and Jellyfin libraries by hand
(they only auto-update on a normal import).

**Requesting an anime grabbed a big pile of downloads.**
A long-running anime makes Sonarr search every episode at once, so it can grab several overlapping
releases (season packs, batches, single episodes). Keep one well-seeded batch and remove the rest in
qBittorrent — using **Interactive Search** to pick a single release avoids it.

**New movies/shows aren't appearing in Plex.**
Make sure you did step E (Plex notifications), and that the Plex library folders match
`C:\Users\YOURNAME\Videos\Movies` and `...\TV Shows`.

**Plex can't see your `C:` drive when adding a library.**
Plex is probably running as a different Windows user. Run Plex as your own user, or give that user
permission to read your Videos folder.

**Seerr can't save its settings (permission error in the logs).**
Run this once to fix the folder ownership, then restart Seerr:
```powershell
docker run --rm -v C:/MediaStack/config/seerr:/data alpine chown -R 1000:1000 /data
```

---

## Sharing this project with someone else

If you give this folder to another person, do **not** include your `.env` file — it holds your own
paths (and any VPN keys). It is already excluded by `.gitignore`, so sharing via Git is safe; if you
zip the folder instead, delete the `.env` copy first. The other person just copies `.env.example` to
`.env` and sets their own `TZ` and `VIDEOS_ROOT` (Part 1, Step 4) — everything else is ready,
including the Claude Code / agent skills.

## Useful commands (run in this folder in PowerShell)

```powershell
docker compose up -d                 # start everything
docker compose stop                  # pause everything
docker compose start                 # resume
docker compose ps                    # see what is running
docker compose logs -f radarr        # watch one app's messages
docker logs qbittorrent              # find qBittorrent's first-time password
docker compose pull; docker compose up -d   # update all apps to the latest version
```
