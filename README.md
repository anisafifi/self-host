# Self-Host

> My personal homelab — built out of curiosity, maintained out of stubbornness.

This is my self-hosting playground. It started as a weekend experiment and slowly turned into something I actually rely on daily. I've broken things, rebuilt them, and learned a lot along the way — and I wouldn't have it any other way.

I care about owning my data and understanding the systems I depend on. This repo is where all of that lives.

---

## Why I built this

I got tired of handing over my data to services I didn't control, paying for things I could run myself, and not truly knowing what was happening under the hood. Self-hosting fixed all of that — and turned out to be genuinely fun.

- **Control** — my data stays on my hardware, on my terms
- **Privacy** — no analytics, no third-party eyes, no surprises
- **Learning** — every broken container taught me something new
- **Simplicity** — one repo, all my stacks, no scattered configs

---

## What's inside

Most folders are standalone services, each with their own `docker-compose.yml` and any related config files they need. A few examples:

| Folder | What it does |
|---|---|
| `adguard/` | Network-wide ad and tracker blocking |
| `jellyfin/` | Personal media server |
| `vaultwarden/` | Self-hosted password manager (Bitwarden-compatible) |
| `immich/` | Google Photos alternative for my own photos |
| `paperless/` | Document management and scanning |
| `uptime-kuma/` | Service monitoring and status pages |
| `authentik/` | SSO and identity provider for everything else |

...and a growing collection of other tools and utilities.

---

## How I run things

Nothing fancy. I go into the service folder, check the config and environment values, then:

```sh
docker-compose up -d
```

That's genuinely it. I like keeping things simple and easy to reason about — no over-engineered orchestration, just composable services that do one thing well.

---

## Before you use any of this

These configs are tailored to my own environment. If you're borrowing something, you'll likely need to adjust:

- **Ports** — I may already be using yours
- **Domains** — swap in your own
- **Volumes** — point to your actual paths
- **Environment variables** — especially secrets, please don't use mine

And as always — **review files before deploying anything publicly**. Don't trust random configs from the internet, including these.

---

## Contributions

If you've got a cleaner way to run something, found a mistake, or just want to suggest an improvement — issues and PRs are welcome. I'm always open to learning a better approach.

---

## License

MIT — use it, fork it, make it your own.