# Security design

What this was built to resist, and how. It's the design level only, because the source is private.

## Context

The server sits between untrusted guests and the internet. It also holds personal files and runs an admin console. The old version leaked an admin password, a guest code and a PIN through its repository, so most of what follows is the opposite of that.

## Threats and controls

| Threat | What stops it |
|---|---|
| Secrets in code or history | Nothing is compiled in. The first admin password is generated on first start, stored only as a bcrypt hash, and handed over once through a root-only file. |
| Stolen or guessed credentials | bcrypt for passwords. Session tokens are stored only as SHA-256 hashes. Sessions expire (12 hours, or 30 days if you ask). Signing out or deleting an account kills its sessions straight away. |
| Brute force | Attempt limits per address on sign-in and on the guest code. A separate per-account limit on password re-confirmation. |
| A signed-in session used for damage | Creating or deleting guest codes or users, and resetting passwords, ask for the admin password again. |
| A user account reaching admin features | Two roles, enforced on the server. A user gets a short allow-list of routes and everything else is refused. A test throws dozens of routes at a user token and expects a refusal on each one that isn't listed. |
| Guests reaching the server | Until a device is authorised it can reach DHCP, DNS and the web server (the portal and the console's sign-in page, which still needs credentials). SSH, file shares and every other service are out of reach. |
| Forged proxy headers | Forwarded-address headers are trusted only from the local proxy, and only the last entry, which the proxy added itself. A hotspot guest can't pass as a tunnel visitor. |
| Command injection | Arguments to system programs are passed without a shell. Interface names, MAC addresses, network ranges and set names are checked against strict patterns first. |
| Path traversal and symlinks | Media is served by an opaque ID, never a path from the browser. Resolved paths have to stay inside the media folder, even through symlinks, and the scan doesn't follow links. |
| Server-side request abuse | Subtitle downloads use only links the server itself got from the service, from that service's domain. Sizes and decompression are capped. |
| Restricted content leaking | Age-gated categories are refused by every route that could expose them, per session, until the session confirms its age. |
| Old and new front-end code mixing after an update | Static files are revalidated on every load. |
| Cross-site requests | State-changing calls need a bearer token, which a foreign page can't send. Cookies are accepted only for read-only media fetches from the same site. |
| Locking yourself out while changing the firewall | Cutover works in steps with backups and an automatic rollback. The SSH port and the dashboard port stay open even with the uplink shield on. |

## What it doesn't claim

- Four-digit guest codes are easy to guess. The attempt limit is the protection, not secrecy.
- A captive portal can't redirect HTTPS. The login sheet depends on plain-HTTP probes.
- It's built for a small private network. It isn't meant for hosting strangers' workloads.
