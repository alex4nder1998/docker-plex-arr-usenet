# Remote access (letting yourself or friends in)

Your stack listens on `localhost` — only reachable from the machine it runs on.
To reach it from your phone, another house, or share **Seerr** with friends, use
one of these. **Do not just port-forward the admin apps** — they have weak/no
auth (see [SECURITY.md](../SECURITY.md)).

Ranked easiest/safest first.

---

## 1. Tailscale (recommended)

A private mesh VPN. Your devices join a network only they can see — **nothing is
exposed to the public internet**, no ports opened, no domain needed. Free for
personal use.

1. Install Tailscale on the **host** (the machine running Docker) and on each
   device that needs access (your phone, a friend's phone). [tailscale.com/download](https://tailscale.com/download)
2. Sign in on each; run `tailscale up` on the host.
3. Find the host's Tailscale IP (`tailscale ip -4`, looks like `100.x.y.z`).
4. Access any service at `http://100.x.y.z:<port>` — e.g. Seerr at
   `http://100.x.y.z:5055`.
5. To share with a friend, use Tailscale's **device sharing** (invite their
   account to just that machine) — they don't join your whole network.

Great for: everything, privately. Best default.

---

## 2. Cloudflare Tunnel (public HTTPS URL, with optional login)

Gives a service a real `https://requests.yourdomain.com` address with a valid
certificate and **no port forwarding**. Add **Cloudflare Access** in front for a
login gate. Requires a domain on Cloudflare (free tier is fine).

1. Add your domain to Cloudflare.
2. Install `cloudflared` on the host and create a tunnel:
   `cloudflared tunnel login` → `cloudflared tunnel create media`.
3. Route a hostname to the local service, e.g. map
   `requests.yourdomain.com` → `http://localhost:5055` (Seerr).
4. **Protect it:** Cloudflare Zero Trust → Access → add an application on that
   hostname → allow only specific emails (yours + friends').

Great for: giving friends a clean Seerr link without them installing anything.

---

## 3. Port forwarding (not recommended)

Opening a router port to an admin app puts an unauthenticated control panel on
the public internet — bots find these fast. If you must expose something,
expose **only** Plex (use Plex's own **Settings → Remote Access**, which is
built for this) or Seerr **behind Cloudflare Access / a reverse proxy with
authentication** — never Radarr/Sonarr/Prowlarr/SABnzbd.

---

## Which services to share?

| Service | Share with friends? | How |
|---|---|---|
| **Plex** | ✅ yes | Plex's built-in Remote Access + library sharing |
| **Seerr** | ✅ yes (so they can request) | Tailscale, or Cloudflare Access |
| Radarr / Sonarr / Prowlarr / SABnzbd / Bazarr | ❌ never | LAN / Tailscale only, admin-only |
