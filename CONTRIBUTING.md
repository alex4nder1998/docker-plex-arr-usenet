# Contributing

Thanks for wanting to improve this stack! It's a community reference — corrections, clearer docs, and real-world fixes are all welcome.

## Ways to help

- **Found a bug or a gotcha?** Open an issue describing your OS, Docker version, and what broke. Real-world failure modes are exactly what this repo tries to document.
- **Have a fix or a clearer explanation?** Open a pull request.
- **Added a service or an alternative** (Jellyfin, a different downloader, etc.)? PRs that keep the single-mount hardlink principle intact are welcome.

## Ground rules

1. **Never commit secrets.** No API keys, passwords, provider logins, or a real `.env`. The `.gitignore` blocks `.env` and `config/` — keep it that way. Use placeholders like `YOUR_RADARR_API_KEY`.
2. **Keep hardlinks working.** Any change must preserve the single `/data` mount (downloads + library on one volume). Splitting them breaks hardlinks — see [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md).
3. **Test before you PR.** Run `docker compose config` to validate the compose file, and ideally spin the affected service up locally.
4. **Document the "why."** If you fix a non-obvious problem, add it to [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) so the next person doesn't hit it.

## Pull request checklist

- [ ] No secrets or personal paths committed
- [ ] `docker compose config` passes
- [ ] Docs updated if behavior changed
- [ ] Change keeps the single-mount / hardlink layout

## Local sanity check

```bash
cp .env.example .env      # fill in test values
docker compose config     # validate
docker compose up -d       # bring it up
docker compose ps          # confirm healthy
```
