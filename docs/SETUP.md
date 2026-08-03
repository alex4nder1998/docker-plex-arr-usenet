# Full Setup Guide (step-by-step, reproducible)

This walks you from nothing to a fully working stack. Every value is spelled out. Do the steps **in order** — later apps depend on earlier ones.

Estimated time: ~30–45 minutes.

---

## 0. Prerequisites

- **Docker + Docker Compose** installed and running
  - Windows/Mac: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
  - Linux: [Docker Engine](https://docs.docker.com/engine/install/) + the `docker compose` plugin
- **A Usenet provider account** (this is the "where files come from"):
  - Cheap unlimited: [Frugal Usenet](https://www.frugalusenet.com) (~$60/yr), Newshosting, Eweka, etc.
- **A Usenet indexer account** (the "search engine"):
  - [NZBGeek](https://nzbgeek.info) (VIP needed for API), NZBFinder, DrunkenSlug, etc.
  - Copy your indexer **API key** — you'll need it in step 4.
- A place to store media with **lots** of free space.

---

## 1. Create the media folder layout

Everything must live under **one** parent folder so hardlinks work. Create these subfolders inside your chosen media root (e.g. `E:\Media` or `/mnt/media`):

```
<media root>/
├── movies/
├── Tv Shows/
├── music/
├── Downloads/       ← completed downloads land here
└── incomplete/      ← in-progress downloads
```

> ⚠️ Do **not** put downloads on a different drive/mount than the library, or you lose hardlinks (files get copied + duplicated instead of instantly linked).

---

## 2. Configure `.env` and start the stack

```bash
git clone https://github.com/alex4nder1998/docker-plex-arr-usenet.git
cd docker-plex-arr-usenet
cp .env.example .env
```

Edit `.env`:
- `DATA_ROOT` → your media root from step 1 (e.g. `E:\Media` or `/mnt/media`)
- `CONFIG_ROOT` → where app configs live (default `./config` is fine)
- `TZ` → your timezone (e.g. `America/New_York`)
- `PUID`/`PGID` → on Linux run `id` and use your user's ids; on Docker Desktop leave `1000`
- `PLEX_CLAIM_TOKEN` → optional, get a fresh one (valid ~4 min) at https://www.plex.tv/claim

Start everything:
```bash
docker compose up -d
```

Give it a minute, then confirm all containers are up:
```bash
docker compose ps
```

**Finding an app's API key later:** open the app → **Settings → General → Security → API Key**.

---

## 3. SABnzbd — the downloader (`http://localhost:8080`)

1. Complete the first-run wizard (skip/no changes are fine).
2. **Config → Servers → Add Server:**
   - Host = your provider's server (e.g. `news.frugalusenet.com`)
   - Port = `563`, **SSL = ✔**
   - Username / Password = your **provider** login
   - Connections = `20–50`
   - **Test Server** → should go green → Save
3. **Config → Folders:**
   - Temporary Download Folder = `/data/incomplete`
   - Completed Download Folder = `/data/Downloads`
4. **Config → Categories** → add three categories named exactly:
   - `radarr`, `sonarr`, `lidarr` (folder = same name)
5. **Config → Switches:** turn **"Abort jobs that cannot be completed"** ON. Set each category's processing to **Repair + Unpack + Delete**. *(Prevents broken downloads from jamming the queue.)*
6. Note SABnzbd's **API key** (Config → General).

---

## 4. Prowlarr — indexers (`http://localhost:9696`)

1. **Settings → General** → note the API key. Set an auth method if prompted.
2. **Indexers → Add Indexer** → find your Usenet indexer (e.g. NZBGeek) → paste its **API key** → Test → Save.
3. **Settings → Apps → Add Application** — add each of Radarr, Sonarr, Lidarr:
   - Prowlarr Server = `http://prowlarr:9696`
   - Radarr/Sonarr/Lidarr Server = `http://radarr:7878` / `http://sonarr:8989` / `http://lidarr:8686`
   - API Key = that app's key
   - Sync Level = **Full Sync** → Test → Save

Your indexer now auto-appears in all three apps.

---

## 5. Radarr / Sonarr / Lidarr

Do these in **each** app (`:7878`, `:8989`, `:8686`).

1. **Settings → Media Management:**
   - **Use Hardlinks instead of Copy = ON** (show Advanced)
   - **Recycling Bin** = `/data/.recycle` (safety net for deletes/upgrades)
   - Rename = ON (recommended)
2. **Settings → Download Clients → Add → SABnzbd:**
   - Host = `sabnzbd`, Port = `8080`, API Key = SABnzbd's key
   - Category = `radarr` / `sonarr` / `lidarr`
   - **Priority ≥ 1** → Test → Save
3. **Root Folder** (Movies/Series/Artist settings or when adding):
   - Radarr = `/data/movies` · Sonarr = `/data/Tv Shows` · Lidarr = `/data/music`
4. *(Recommended)* Add the **"No English Audio"** custom format — see [custom-formats.md](custom-formats.md).

---

## 6. Plex (`http://localhost:32400/web`)

1. Sign in / claim the server.
2. **Add Library** for each: Movies → `/data/movies`, TV Shows → `/data/Tv Shows`, Music → `/data/music`.
3. **Settings → Network → LAN Networks** → add `172.20.0.0/16` (and your real LAN, e.g. `192.168.0.0/24`). *(Stops Plex from treating LAN clients as remote and transcoding — see [custom-formats.md](custom-formats.md).)*

---

## 7. Seerr — request page (`http://localhost:5055`)

1. **Sign in with Plex** (OAuth in the browser).
2. Add your Plex server → select the libraries.
3. **Settings → Services → Radarr / Sonarr** → Add:
   - Server = `http://radarr:7878` / `http://sonarr:8989`, API key, default quality profile + root folder
   - **Leave "Tag Requests" OFF** *(with it on, an empty tag makes Radarr reject the request with a 400).*

Now users can request movies/TV, and Seerr hands them to Radarr/Sonarr → SABnzbd → Plex.

---

## 8. Bazarr — subtitles (`http://localhost:6767`)

1. **Settings → Sonarr / Radarr** → Address `sonarr` / `radarr`, port, API key → Test → Save.
2. **Settings → Providers** → enable a few (OpenSubtitles.com free account works well).
3. **Settings → Languages** → add your language → create a Language Profile → set it as the default for Series & Movies.

---

## 9. Homepage dashboard (`http://localhost:3000`)

Copy `examples/homepage/services.yaml` to `${CONFIG_ROOT}/homepage/services.yaml` and paste each app's API key in. Reload the page → live widgets for downloads, library counts, etc.

---

## Done ✅

Request something in Seerr and watch it flow: **search → download → hardlink → Plex.** If anything misbehaves, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md).
