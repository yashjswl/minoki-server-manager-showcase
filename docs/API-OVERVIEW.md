# HTTP API overview

Everything in the console is an API call, and the command-line tool uses the same API over a root-only unix socket. The product documents each route (request fields, example responses and notes) on its own Documentation page. This is the short version.

## Conventions

Requests and responses are JSON. You sign in once, receive a bearer token, and send it with every other call.

Success is `{"success": true}`, or the new state, such as `{"enabled": true}`. Errors are `{"error": "..."}` with a status code that means something: 400 for bad input, 401 when signed out, 403 when not allowed (or when the re-confirmed password was wrong), 404, 409 for a conflict, 429 when rate limited, and 502 when an upstream service failed.

Responses aren't cached. Sensitive changes take an extra `admin_password` field. There are two roles. A user account has a short allow-list and receives a 403 for everything else.

## Route families

| Family | Routes | What's in it |
|---|---|---|
| Authentication and account | 4 | Sign in and out, who am I, change my own password |
| Status and monitoring | 10 | System meters, throughput per interface, top applications, disks, fans, service health, activity feed, connectivity and public address, speed test |
| Network control | 23 | Internet sharing, captive portal and session length, guest access codes (list, create, delete), user management (list, create, reset, delete), VPN, uplink and Wi-Fi, interfaces and switching adapters on or off, hotspot band and channel, settings, hotspot restart |
| Clients and devices | 7 | Devices on the network, names, blocking, speed limits, manual authorisation |
| Logs, alerts and power | 7 | Blocked-connection log, sound and voice settings, a free-text note, reboot and shutdown |
| Videos | 31 | Library, titles, streaming, posters, subtitles, progress, episode management, extra audio tracks, settings, artwork refresh |
| Guest portal | 4 | Code verification, "am I authorised", a device's own address, and a tunnel-aware admin authorisation |
| Total | 86 | |

## Examples

Signing in:

```http
POST /api/login
{"user": "admin", "password": "••••••••", "remember": true}

200 {"token": "…", "role": "admin"}
```

System status:

```http
GET /api/status
Authorization: Bearer …

200 {
  "system":  {"cpu": 21, "ram": 45, "disk": 27, "uptime": "4h 34m", "temp": 58, "hostname": "minoki-server"},
  "sharing": true, "captive": true, "vpn": false, "uplink": "wlp1s0",
  "hotspot": {"ssid": "Minoki Guest", "interface": "wlan1"}
}
```

Creating a guest access code (the password is asked for again):

```http
POST /api/portal/codes
{"label": "Workshop attendees", "code": "", "password": "••••••••"}

200 {"code": {"id": "…", "label": "Workshop attendees", "code": "7305", "uses": 0, "last_used": 0}}
```

A user account calling an admin route:

```http
GET /api/clients
Authorization: Bearer <a user's token>

403 {"error": "This account cannot do that."}
```
