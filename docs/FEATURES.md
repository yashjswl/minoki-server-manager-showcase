# Feature tour

A walk through the product, area by area. Screenshots use invented sample data.

## 1. The gateway

The server shares one internet connection (Wi-Fi or a wired port) with devices on its own hotspot.

- **Internet sharing** is a master switch. Off means the hotspot cannot reach the internet at all (a kill switch).
- **Uplink selection** between the Wi-Fi and wired ports, with Wi-Fi scan and connect, and a clear warning when the
  uplink Wi-Fi sits behind its own login page (a plain ping would wrongly say "online").
- **Per-device control:** rename, block, or limit a device's speed. Limits survive reboots.
- **Guest isolation:** devices that have not signed in can reach only DHCP, DNS and the web server.
- **WAN shield:** drops unsolicited inbound traffic on the uplink while always keeping the management ports open, so you
  cannot lock yourself out.

![Network](../assets/screenshots/network.png)

## 2. The guest portal

- Guests open any web page and are shown a sign-in page; phones open their own sign-in sheet automatically.
- The administrator keeps **several four-digit codes** at once (for family, for a weekend, for a workshop), sees how often
  each was used, and deletes one at any time. Adding or deleting asks for the administrator's password again.
- Sessions expire after a length you choose (30 minutes to 24 hours). One button signs every guest out.

![Access codes](../assets/screenshots/hotspot-access-codes.png)

## 3. Monitoring and health

- **Overview:** the path to the internet (uplink, gateway, hotspot, clients) with real health and live traffic per link,
  then throughput, system meters, VPN state, clients, top applications and recent events.
- **System:** per-resource detail, disks, interfaces, and power actions behind a typed confirmation.
- **Services:** the self-hosted apps (files, music, photos, videos, DNS filter, monitoring, web terminal) with live status.
- **Alerts and sound:** spoken and audible alerts for power, VPN and intrusion events, with a language choice and a
  volume that can boost past the hardware maximum.
- **Logs:** blocked connections, with a switch to hide routine local-network chatter.

![Overview](../assets/screenshots/overview-light.png)

## 4. The video library

A streaming-style library built into the daemon and the console.

- Folders become categories; a folder of episodes becomes a series; loose files named like episodes group into a series
  automatically.
- Posters from a movie database when a key is set, otherwise a frame from the video. Titles can be renamed, regrouped,
  hidden and renumbered without touching the files.
- Playback adapts to the file: direct play when the browser supports it, otherwise a one-time background repackage into
  MP4 (with honest progress, including the slow final pass), otherwise an external-player link.
- Subtitles from embedded streams, files next to the video, or an online search, with a timing nudge when they are out
  of sync. Watch progress is per user, with "continue watching".
- Restricted (A-rated) categories stay hidden until a session confirms its age, and the server enforces it.

![Player](../assets/screenshots/videos-player.png)

## 5. Accounts and consoles

- The administrator creates **user** accounts. A user signs in on the same page and gets a smaller console:
  Overview (without the administrator's tools), Services (five apps), Videos, and Settings.
- The server, not the page, refuses everything else.

![Sign in](../assets/screenshots/login.png)

![User console](../assets/screenshots/user-overview.png)

![Settings in dark theme](../assets/screenshots/settings-dark.png)

## 6. Installable apps

The console and the Video library are separate progressive web apps, each with its own icon, name and startup screen, so
they sit on a phone's home screen like native apps. Layouts differ for desktop (sidebar, dense tables) and phone (bottom
bar, list rows, bottom sheets).

![Phone](../assets/screenshots/phone-overview.png)

## 7. Operating it

- **Provisioning** installs and configures the supporting services, adopts what already exists, and backs up any file it
  rewrites. A dry run shows the plan first.
- **Cutover** moves a live gateway from the old system with backups, per-step verification, and automatic rollback.
- **Doctor** reports the health of every managed service from the command line or the console.
- **Documentation** is part of the product, and a script checks that no route is left undocumented.
