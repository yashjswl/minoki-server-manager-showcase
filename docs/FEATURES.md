# Feature tour

Area by area. The screenshots use invented sample data.

## 1. The gateway

The server shares one internet connection (Wi-Fi or a wired port) with the devices on its own hotspot.

Internet sharing is a master switch. Off means the hotspot can't reach the internet at all, which doubles as a kill switch.

You can pick the uplink between the Wi-Fi and wired ports, scan for networks and connect. If the uplink Wi-Fi sits behind its own login page, the console says so. A plain ping would report "online" there and be wrong.

Per device, you can rename, block or limit speed. Limits survive reboots.

Until a device signs in, it can reach DHCP, DNS and the web server and nothing else. There's also an uplink shield that drops unsolicited inbound traffic. It always leaves the management ports open, so you can't lock yourself out with it.

![Network](../assets/screenshots/network.png)

## 2. The guest portal

Guests open any web page and land on a sign-in page. Phones are redirected so their own login sheet opens.

The admin keeps several four-digit codes at once. One for family, one for a weekend, one for a workshop, say. Each shows how often it's been used, and any of them can be deleted at any time. Adding or deleting asks for the admin password again.

Sessions last as long as you set, from 30 minutes to 24 hours. One button signs every guest out.

![Access codes](../assets/screenshots/hotspot-access-codes.png)

## 3. Monitoring and health

The Overview shows the path to the internet (uplink, gateway, hotspot, clients) with real health and live traffic on each link. Below it: throughput, system meters, VPN state, clients, top applications and recent events.

System has the per-resource detail, disks, interfaces, and power actions that need a typed confirmation. Services lists the self-hosted apps with live status. Logs shows blocked connections, with a switch to hide the routine chatter from other devices on the network.

Alerts can be spoken or played as sounds, for power, VPN and intrusion events. You pick the language and the volume, and the volume can go past the hardware maximum.

![Overview](../assets/screenshots/overview-light.png)

## 4. The video library

It's built into the daemon and the console.

Folders become categories. A folder of episodes becomes a series, and loose files named like episodes get grouped into one automatically. Posters come from a movie database if you've set a key, and from a frame of the video if you haven't. You can rename, regroup, hide and renumber titles without touching the files.

How a file plays depends on the file. If the browser can play it, it plays. Otherwise it's repackaged to MP4 once in the background, with a progress bar that also covers the slow last pass. If that won't work either, you get a link for an external player.

Subtitles come from embedded streams, files next to the video, or an online search. There's a timing nudge for when they're out of sync. Watch progress is per user.

Restricted (A-rated) categories stay hidden until a session confirms its age, and the server enforces that.

![Player](../assets/screenshots/videos-player.png)

## 5. Accounts and consoles

The admin creates user accounts. A user signs in on the same page and gets a smaller console: Overview without the admin tools, five apps under Services, Videos, and Settings. The server refuses everything else, whatever the page shows.

![Sign in](../assets/screenshots/login.png)

![User console](../assets/screenshots/user-overview.png)

![Settings in dark theme](../assets/screenshots/settings-dark.png)

## 6. Installable apps

The console and the video library are separate progressive web apps. Each has its own icon, name and startup screen, so they sit on a phone's home screen like native apps. Desktop gets a sidebar and dense tables. Phone gets a bottom bar, list rows and bottom sheets.

![Phone](../assets/screenshots/phone-overview.png)

## 7. Operating it

Provisioning installs and configures the supporting services, adopts what's already installed, and backs up any file it rewrites. A dry run shows the plan first.

Cutover moves a live gateway over from the old system, with backups, a check after each step, and an automatic rollback.

Doctor reports on every managed service, from the command line or the console.

The documentation lives inside the product. I check with a small script that no route is left out.
