# Security

This repository is configuration and documentation only — it ships no code that
processes untrusted input. The security that matters here is **how you run the
stack**. A misconfigured *arr stack is the #1 way people get burned.

## Reporting

- **A problem with this repo** (a leaked secret in an example, a broken/insecure
  default): open a GitHub issue or PR.
- **A vulnerability in one of the apps** (Sonarr, SABnzbd, etc.): report it to
  that project upstream, not here.

## Protect your instance

- **Never commit `.env` or `config/`.** Both are in `.gitignore`. They contain
  API keys, provider logins, and tokens. Never paste real API keys into issues
  or screenshots.
- **Do not expose the admin UIs to the internet.** Radarr, Sonarr, Lidarr,
  Prowlarr, SABnzbd, Bazarr, and Homepage have weak or no authentication by
  default. Anyone who can reach `:7878`, `:8080`, etc. can control your
  downloads and **read your API keys**. Keep them on your LAN.
- **Turn on app authentication.** In each *arr app: Settings → General →
  Authentication → **Forms (Login Page)**, and set it to be required. Give
  SABnzbd a username/password too.
- **For remote access, don't port-forward — tunnel.** Use Tailscale or a reverse
  proxy with authentication. See [docs/remote-access.md](docs/remote-access.md).
  The only services designed to face the public are **Plex** (it has its own
  account auth) and, behind an auth layer, **Seerr**.
- **Rotate leaked keys.** If an API key is exposed, regenerate it in that app's
  Settings → General → Security.
- **Keep images updated:** `docker compose pull && docker compose up -d`.
