# Troubleshooting

Real problems this stack hits and exactly how to fix them.

---

### Downloads work, but movies/episodes never appear (and files are duplicated)
**Cause:** downloads and the library are on **different mounts**, so the *arr app copies instead of hardlinking — wasting space and often failing.
**Fix:** everything (Downloads, incomplete, movies, Tv Shows, music) must be under the **one** `${DATA_ROOT}` volume, mounted as `/data`. Verify hardlinks work from inside a container:
```bash
docker exec radarr sh -c 'touch /data/Downloads/.t && ln /data/Downloads/.t /data/movies/.t && stat -c "links=%h" /data/movies/.t && rm -f /data/Downloads/.t /data/movies/.t'
# links=2  ==> hardlinks work
```

---

### *arr shows "download client SABnzbd ... directory does not appear to exist"
**Cause:** the category folder (`/data/Downloads/<category>`) doesn't exist yet.
**Fix:** create `Downloads/radarr`, `Downloads/sonarr`, `Downloads/lidarr` under your media root. The warning clears on the next health check.

---

### *arr can't reach SABnzbd — "Unable to connect ... 403 Forbidden"
**Cause:** SABnzbd's `host_whitelist` doesn't include the container name.
**Fix:** SABnzbd → Config → Special → add `sabnzbd` (and `localhost`) to **Host whitelist**, or clear it. Restart SABnzbd.

---

### Queue full of stuck "completed / importBlocked — No files eligible for import"
**Cause:** a Usenet download had missing articles; SABnzbd left **0-byte files** but marked the job Completed.
**Fix (prevent):** SABnzbd → **"Abort jobs that cannot be completed" = ON**, categories = **Repair+Unpack+Delete**. Broken jobs now fail so the *arr app blocklists + retries a good release.
**Fix (clean up existing):** in the *arr queue, remove the stuck items with **Blocklist + Remove from client**, then re-search.

---

### It grabs foreign-audio (Spanish/etc.) releases
**Cause:** no language rule — the "best" release wins even if it's foreign-only.
**Fix:** add the **"No English Audio"** custom format (Language = English, **Negate ON**) scored `-1000` in your quality profile. See [custom-formats.md](custom-formats.md).

---

### Plex buffers/struggles on LAN even on gigabit ethernet
**Cause 1 (main):** in Docker's bridge network, Plex sees every client as the gateway IP `172.20.0.1` and treats your **LAN** device as **remote → transcodes**.
**Fix:** Plex → Settings → Network → **LAN Networks** → add `172.20.0.0/16`.
**Cause 2:** the file is a **Remux** with **TrueHD/Atmos** audio that clients like Apple TV can't Direct-Play, forcing a CPU transcode (no HW transcode in Docker Desktop).
**Fix:** prefer **WEB-DL** over Remux, or set a Dolby Digital (AC3/EAC3) track as the default audio.

---

### Seerr container crash-loops: `EACCES: permission denied, open '/app/config/logs/...'`
**Cause:** Seerr runs as the `node` user but can't write to the bind-mounted config on Docker Desktop.
**Fix:** add `user: "0:0"` to the `seerr` service (already in this compose) and recreate: `docker compose up -d --force-recreate seerr`.

---

### Movie requests in Seerr/Overseerr silently fail (TV works)
**Cause:** the Radarr service has **"Tag Requests" ON**, but an empty tag label is sent and Radarr rejects empty tags (`400 Failed to create tag`).
**Fix:** Seerr → Settings → Services → Radarr → turn **Tag Requests OFF**. Retry the failed request.

---

### Migrating from Overseerr/Jellyseerr
Point the `seerr` service at your **existing** Overseerr/Jellyseerr config folder (`:/app/config`). Seerr auto-migrates on first start ("migration completed successfully"). Back up the config folder first.

---

### Useful commands
```bash
docker compose ps                 # what's running
docker logs <name> --tail 50      # a service's logs
docker compose pull && docker compose up -d   # update everything
docker compose restart <name>     # restart one service
```
