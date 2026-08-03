# Custom Formats & key settings

## "No English Audio" (reject foreign-audio releases)

Radarr/Sonarr will happily grab a release with only Spanish/foreign audio if it's the "best" match. This custom format rejects any release that does **not** contain English audio.

Create it in **Settings → Custom Formats → Add** (Radarr shown; Sonarr identical):

- **Name:** `No English Audio`
- **Condition → Language:**
  - Language = `English`
  - **Negate = ON** (matches releases that do NOT have English)

Then in **Settings → Profiles → your Quality Profile**, set that custom format's **score to `-1000`** and leave **Minimum Custom Format Score = 0**. Any release lacking English audio scores below the minimum → rejected. Multi-language releases that include English are unaffected.

> Swap `English` for your own language as needed.

## SABnzbd — don't let broken downloads jam the queue

By default a Usenet download with missing articles can finish as "Completed" with 0-byte files, which then stick in the *arr queue as `importBlocked`. Fix it:

- **Config → Switches → "Abort jobs that cannot be completed" = ON** (`fail_hopeless_jobs`)
- Each category's post-processing = **Repair + Unpack + Delete**

Now incomplete jobs **fail**, and Sonarr/Radarr automatically blocklist the bad release and search for a working one.

## Plex — Direct Play on your LAN (Docker gotcha)

Because Plex runs in a bridge network, it sees every client as coming from the Docker gateway (`172.20.0.1`) and treats your LAN devices as **remote**, forcing transcoding.

**Fix:** Plex → Settings → Network → **LAN Networks** → add your Docker subnet: `172.20.0.0/16` (and your real LAN, e.g. `192.168.0.0/24`).

Also: clients like Apple TV can't Direct-Play lossless **TrueHD/Atmos** — for smooth streaming prefer **WEB-DL** over Remux, or set a Dolby Digital (AC3/EAC3) audio track as default.
