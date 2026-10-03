# Security design

A summary of the threats this project was designed against and how. (The source is private; this is the design level.)

## Context

The server sits between untrusted guests and the internet, holds personal files and media, and exposes an admin
console. The previous version exposed an admin password, a guest code and a PIN in its repository. Everything below
follows from not repeating that.

## Controls

| Threat | Control |
|---|---|
| **Secrets in code or history** | Nothing is compiled in. The first administrator password is generated at first start, stored only as a bcrypt hash, and shown once through a root-only file. |
| **Stolen or guessed credentials** | bcrypt for passwords; session tokens stored only as SHA-256 hashes; sessions expire (12 hours, or 30 days by choice); signing out and deleting an account revoke sessions at once. |
| **Brute force** | Per-address limits on sign-in and on the guest code, and a separate per-account limit on password re-confirmation. |
| **A signed-in session used for harm** | Creating or deleting guest codes and user accounts, and resetting passwords, ask for the administrator's password again. |
| **A user account reaching admin features** | Two roles, enforced on the server. A user may call a short allow-list of routes; anything not listed is refused. A test checks dozens of routes, including ones that do not exist yet. |
| **Guests reaching the server itself** | Until a device is authorised it can reach only DHCP, DNS and the web server (the portal and the console's sign-in page, which still needs credentials). SSH, file shares and every other service are not reachable. |
| **Forged proxy headers** | Forwarded-address headers are trusted only from the local proxy, and only the last entry, which the proxy itself appended. A hotspot guest cannot pretend to be a tunnel visitor. |
| **Command injection** | Arguments to system programs are passed without a shell, and interface names, MAC addresses, network ranges and set names are validated against strict patterns before use. |
| **Path traversal and symlinks** | Media is served by an opaque ID, never a path from the browser. Resolved paths must stay inside the media folder even through symbolic links; links are not followed by the scan. |
| **Server-side request abuse** | Downloads (subtitles) use only links the server itself received from the service, and only from that service's domain. Sizes and decompression are capped. |
| **Restricted-content leakage** | Age-gated categories are refused by every route that could expose them, per session, until the session confirms its age. |
| **Stale or mixed front-end code after an update** | Static files are revalidated on every load. |
| **Cross-site requests** | State-changing calls need a bearer token that a foreign page cannot send. Cookies are accepted only for read-only media fetches from the same site. |
| **Lock-out while changing the firewall** | Cutover is staged with backups and an automatic rollback; the SSH port and the dashboard port stay open even with the WAN shield on. |

## What it deliberately does not claim

- Four-digit guest codes are easy to guess; the protection is the attempt limit, not secrecy.
- HTTPS sites cannot be redirected by a captive portal, so the sign-in sheet relies on plain-HTTP probes.
- It is built for a small private network, not for hostile multi-tenant hosting.
