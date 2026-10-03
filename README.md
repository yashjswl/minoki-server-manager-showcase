# Minoki Server Manager

A Go daemon, a command-line tool and a web console that turn one small Linux box into a Wi-Fi gateway with a guest login page, a VPN, monitoring and a video library.

![Overview](assets/screenshots/overview-light.png)

This repo is only the write-up: screenshots, diagrams and notes. The source is private and isn't licensed for reuse (see [LICENSE](LICENSE)). If you want to read it, ask. Contact details are at the bottom. Every name, device, address and video in the screenshots is made up.

![A tour of the console](assets/demo.gif)

## Why it exists

The first version was a pile of PHP pages that ran `sudo` on bash scripts. Those scripts handled the hotspot, the guest login, the firewall, the VPN and a few self-hosted apps. Passwords and a PIN sat in the code. There were no tests, so every change was a guess about what a script would do to the firewall.

I replaced it with one program that owns the firewall and everything around it, with a single authenticated API in front. It's in production on a 32-bit Ubuntu 18.04 machine with a Core 2 Duo and 2 GB of RAM. The box is old and small, and that shaped a lot of the decisions below.

## What it does

It shares an internet connection with devices on its own hotspot, and it controls who gets through.

- Firewall rules, applied in one go. Per-device blocking and speed limits, a master kill switch, and a shield for the uplink side.
- A guest portal. Unknown devices get a sign-in page, and phones are redirected so their login sheet opens on its own. The admin keeps several four-digit codes at once and sees how often each is used.
- WireGuard VPN with a watchdog, plus switching between a Wi-Fi uplink and a wired one. It also spots uplinks that sit behind their own login page, which a plain ping reports as "online".
- Live monitoring: traffic per interface, top processes, meters, service health, an activity feed. Alerts can be spoken out loud.
- A provisioner that installs about fifteen services, adopts the ones already there instead of overwriting them, and can take over from the old setup with a rollback.
- A video library with posters, series, resume, subtitles and one-time MKV repackaging.
- Two consoles behind one login: the full admin one, and a smaller one for ordinary users.
- Installable phone apps. The console and the video library are separate PWAs.

## Running it without breaking it

You can't run this from here, since there's no source. But the way it gets operated is part of the design, so here's the short version.

Anything that touches the system has a dry run first. Provisioning prints a plan. Migration shows what it would import. Cutover has a preflight step.

Cutover backs up the old setup, checks each step, and rolls itself back if you don't confirm within a few minutes. That's the part that lets you change a live gateway over SSH without locking yourself out.

The daemon owns its own firewall chains and loads them in a single `iptables-restore`. Apply them twice and nothing changes. Restart the daemon and it reapplies everything from its database.

One rule of thumb: don't switch the uplink from a connection that depends on the old uplink. Use a cable or the hotspot side.

## Screenshots

| | |
|---|---|
| ![Clients](assets/screenshots/clients.png) | ![Network](assets/screenshots/network.png) |
| Clients, with block and speed limit | Network: uplink, sharing, speed test |
| ![Hotspot access codes](assets/screenshots/hotspot-access-codes.png) | ![Settings and user management](assets/screenshots/settings-user-management.png) |
| Guest access codes | User management |
| ![Services](assets/screenshots/services.png) | ![System](assets/screenshots/system.png) |
| Services and their health | System meters and power |
| ![Alerts](assets/screenshots/alerts.png) | ![Logs](assets/screenshots/logs.png) |
| Alerts and sound | Blocked connections |

The video library:

| | |
|---|---|
| ![Library home](assets/screenshots/videos-home.png) | ![Series and episodes](assets/screenshots/videos-series.png) |
| Home, with continue watching | A series and its episodes |
| ![Player](assets/screenshots/videos-player.png) | ![Episode management](assets/screenshots/videos-manage.png) |
| Player, with subtitle timing | Edit, hide, regroup |

Dark theme, phone layouts and the guest portal:

| | | | |
|---|---|---|---|
| ![Dark](assets/screenshots/overview-dark.png) | ![Phone overview](assets/screenshots/phone-overview.png) | ![Phone videos](assets/screenshots/phone-videos.png) | ![Guest portal](assets/screenshots/phone-portal.png) |
| Dark | Phone | Phone videos | Guest portal |

The user console is the same app with fewer pages and none of the admin tools:

| | | | |
|---|---|---|---|
| ![User overview](assets/screenshots/user-overview.png) | ![User services](assets/screenshots/user-services.png) | ![User videos](assets/screenshots/user-videos.png) | ![User on a phone](assets/screenshots/phone-user-overview.png) |
| Overview | Services | Videos | Phone |

The console documents itself, including every HTTP endpoint with examples. I check with a small script that no route is missing.

![Documentation](assets/screenshots/api-reference.png)

## Architecture

```mermaid
flowchart LR
  subgraph Clients
    B["Browser or phone app (PWA)"]
    G["Guest phone"]
    C["minoki CLI"]
  end
  subgraph Server["One small Linux server"]
    N["nginx"]
    subgraph D["minokid: one static Go binary"]
      API["HTTP API, sessions, roles"]
      P["Captive portal"]
      M["Media library"]
      F["Firewall engine"]
      S["Supervisor and provisioner"]
      W["Watchdogs and alerts"]
    end
    DB[("SQLite")]
    K["iptables, ipset, tc, WireGuard"]
    A["Hotspot, DNS, files, music, photos"]
  end
  B --> N --> API
  G --> N
  C -- "root-only unix socket" --> API
  API --> DB
  P --> F
  F --> K
  S --> A
  W --> K
  M --> DB
```

More in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Decisions I'd talk about in an interview

The firewall is never half-applied. Rules go into chains the program owns and load in one `iptables-restore`. If it fails, the old rules stay. Golden-file tests cover the generated rules.

Roles are deny-by-default. A user account can call a short list of routes and nothing else, so a new admin route is closed to users the moment it exists. A test fires dozens of routes at a user token and expects a 403 on every one that isn't on the list. The console hides what a user can't use, but the server is what refuses.

Sensitive changes ask for the admin password again, even when you're signed in. Creating or deleting a guest code or a user does this, with its own attempt limit.

Getting a phone to open its login sheet is fiddly. Phones decide by fetching a known web address. The server sends unauthorised guests' plain-HTTP traffic to the portal, and after sign-in it sends the phone back to that address so the sheet closes.

On a Core 2 Duo, transcoding video isn't an option. So the library repackages MKV to MP4 once, in the background, at the lowest priority, and caches the result. The binaries are static with no libc dependency, so the old distro's libraries don't matter.

Age-gated categories are refused by every route that could leak them (titles, artwork, subtitles, files, direct links) until a session confirms its age. Hiding them in the UI isn't the protection.

The console has no build step. Plain ES modules, CSS variables for the light and dark themes, a hand-drawn icon set, and separate layouts for desktop and phone. No framework, no bundler, no CDN requests.

### Bugs that taught me something

- A progress bar stuck at 0%. The server's old ffmpeg reports elapsed time under a different key than new versions do.
- A console that wouldn't start after an update. The browser paired a new script with a cached old one. Now the server tells browsers to revalidate every time.
- An nginx directive that isn't allowed inside `if`. I found it by testing the config before reloading.
- A progress bar stuck at 99%. It was a minutes-long final pass over a 1.5 GB file. The player now says "Finishing up".

## The numbers

| | |
|---|---|
| Go source | about 17,400 lines, 2 programs, 16 internal packages |
| Go tests | about 7,700 lines, 266 passing |
| Console JavaScript | about 4,100 lines, no framework |
| HTTP API | 81 documented endpoints, all authenticated and role-checked |
| Runs on | 32-bit Ubuntu 18.04 in production |
| Dependencies | Go standard library, a pure-Go SQLite driver, bcrypt |

## Stack

Go, SQLite, iptables, ipset, tc, WireGuard, hostapd, dnsmasq, nginx, ffmpeg, systemd. The front end is plain JavaScript and CSS, delivered as PWAs. Visual checks run in headless Chrome.

## Read next

- [Feature tour](docs/FEATURES.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Security design](docs/SECURITY.md)
- [API overview](docs/API-OVERVIEW.md)

## Contact

From Yashasvi Jaiswal. Source code is available for review on request.

LinkedIn: [linkedin.com/in/yashjswl](https://www.linkedin.com/in/yashjswl/)

---

&copy; 2026 Yashasvi Jaiswal. All rights reserved.
