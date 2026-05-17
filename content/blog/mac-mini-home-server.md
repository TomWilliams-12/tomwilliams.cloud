---
title: "Turning a Mac mini Into a Home Server for Self-Hosted Services"
date: 2026-05-17
draft: false
tags: ["homelab", "self-hosting", "mac-mini", "docker", "tailscale"]
---

The Mac mini is one of the more underrated pieces of homelab hardware you can buy. It's small, near-silent, sips power, and ships with an absurd amount of CPU per watt thanks to Apple Silicon. I've been running mine as a 24/7 home server for a while now, and this post is the writeup I wish I'd had when I started.

## Why a Mac mini and not a Raspberry Pi or a NUC?

The honest answer: I already had one sitting on a shelf. But there are a few reasons it turned out to be a great fit:

- **Performance per watt.** An M-series Mac mini will idle around 4–7W and still rip through container workloads that would make a Pi cry.
- **It's silent.** No fans spinning up when Plex transcodes or when a backup job kicks off.
- **macOS is a real Unix.** You get Homebrew, launchd, ZFS via OpenZFS if you really want it, and a familiar shell environment.
- **It just stays up.** Mine has been running for months without a reboot beyond the occasional OS update.

The trade-offs are real though — you're paying Apple prices, you don't get ECC RAM, and storage expansion means hanging things off USB or Thunderbolt. If you need 40TB of spinning rust, build a NAS. For everything else, the Mac mini is excellent.

## Initial macOS setup

A few settings to change before anything else:

**Disable sleep.** In System Settings → Energy:
- Prevent automatic sleeping when the display is off: **on**
- Start up automatically after a power failure: **on**
- Wake for network access: **on**

**Enable auto-login** so the machine comes back up unattended after a power blip. Yes, this trades some security for uptime — make sure the box lives somewhere physically safe.

**Turn on Remote Login (SSH)** in System Settings → General → Sharing. While you're there, give the machine a sensible hostname.

**Install the command line tools** and Homebrew:

```bash
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Container runtime: OrbStack

I run almost everything in containers, and on Apple Silicon, **OrbStack** beats Docker Desktop handily. It's faster to start, uses less RAM, has better filesystem performance, and the networking just works.

```bash
brew install --cask orbstack
```

Set it to start at login and you're done. The `docker` and `docker compose` CLIs work exactly as you'd expect.

## Reverse proxy: Caddy

For routing traffic to services and handling TLS, Caddy is the path of least resistance. A single `Caddyfile` and you have automatic HTTPS from Let's Encrypt for every service.

```caddy
jellyfin.home.example.com {
    reverse_proxy localhost:8096
}

homeassistant.home.example.com {
    reverse_proxy localhost:8123
}
```

I run Caddy in a container alongside everything else, with the config mounted from `~/srv/caddy/Caddyfile`.

## Remote access without opening ports: Tailscale

This is the single most important piece of the setup. **Do not** port-forward your home router to expose services to the internet. Instead, install Tailscale on the Mac mini and on every device you want to reach it from:

```bash
brew install --cask tailscale
```

Now your Mac mini has a stable `100.x.x.x` IP that's only reachable by your own devices. Combine it with [MagicDNS](https://tailscale.com/kb/1081/magicdns/) and you can hit `http://mac-mini:8123` from your phone, anywhere in the world, with no firewall changes.

For services I want family members to reach, Tailscale's Funnel feature exposes a single service to the public internet through their edge, with TLS handled for you.

## The services I actually run

Here's the current lineup, all in Docker Compose:

| Service | What it does |
|---|---|
| **Jellyfin** | Media server. Hardware transcoding works on Apple Silicon with the right flags. |
| **Home Assistant** | Smart home brain. Runs in a container, talks to Zigbee via a USB stick. |
| **Pi-hole** | Network-wide ad blocking. DNS for the whole house points here. |
| **Paperless-ngx** | OCR'd document archive. Scan once, search forever. |
| **Vaultwarden** | Self-hosted Bitwarden-compatible password manager. |
| **Uptime Kuma** | Tells me when something I forgot about has fallen over. |
| **Syncthing** | Folder sync across all my machines without going through the cloud. |

Each one is a folder in `~/srv/` with its own `docker-compose.yml` and persistent volumes underneath. Boring, predictable, easy to back up.

## Storage and backups

My internal SSD holds the OS, container images, and small databases. For media and bulk data I have a Thunderbolt enclosure with a couple of SSDs in it, mounted at `/Volumes/data`.

For backups I run two layers:

1. **Time Machine** to a separate external drive, for the whole system.
2. **Restic** to Backblaze B2 for the irreplaceable stuff — Vaultwarden, Paperless, Home Assistant configs, photos. Encrypted, deduplicated, cheap.

A nightly `launchd` job kicks off the Restic run and pings a healthcheck URL when it succeeds. If the ping doesn't arrive, Uptime Kuma yells at me.

## Monitoring

Nothing fancy: **Uptime Kuma** for "is the service responding," and **Beszel** for lightweight host metrics (CPU, RAM, disk, container health). Both run in containers, both have web UIs I can hit over Tailscale.

## What I'd do differently

- **Buy the bigger SSD.** Container images, databases, and Time Machine snapshots eat space faster than you think.
- **Get the Thunderbolt enclosure sooner.** USB 3 is fine until you're moving real volumes of data.
- **Document the compose files in a git repo from day one.** Past-me thought he'd remember which env vars he set. Past-me was wrong.

## Wrapping up

A Mac mini won't replace a rack of Dell servers, but for a home setup that quietly hosts a dozen useful services and disappears into a shelf, it's hard to beat. The combination of low power draw, silent operation, and a real Unix environment makes it a genuinely lovely machine to run things on.

If you've got one gathering dust, give it a job.
