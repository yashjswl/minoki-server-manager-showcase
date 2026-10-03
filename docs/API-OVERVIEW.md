# HTTP API overview

Everything in the console is an API call, and the same API is available on a root-only unix socket for the
command-line tool. The product documents every route with request fields, example responses and notes on its own
Documentation page; this is the shape of it.

## Conventions

- JSON in, JSON out. Sign in once for a bearer token, send it on every other call.
- Success is `{"success": true}` or the new state (for example `{"enabled": true}`). Errors are `{"error": "..."}` with a
  meaningful status: `400` bad input, `401` signed out, `403` not allowed (or wrong re-confirmed password), `404`, `409`
  conflict, `429` rate limited, `502` an upstream service failed.
- Responses are never cached. Sensitive changes take an extra `admin_password` field.
- Two roles. A user gets a short allow-list and a `403` everywhere else.

## Route families

| Family | Routes | What it covers |
|---|---|---|
| Authentication and account | 4 | Sign in and out, who am I, change my own password |
| Status and monitoring | 10 | System meters, throughput per interface, top applications, disks, fans, service health, activity feed, connectivity and public address, speed test |
| Network control | 20 | Internet sharing, captive portal and session length, guest access codes (list, create, delete), user management (list, create, reset, delete), VPN, uplink and Wi-Fi, interfaces, settings, hotspot restart |
| Clients and devices | 7 | Devices on the network, names, blocking, speed limits, manual authorisation |
| Logs, alerts and power | 7 | Blocked-connection log, sound and voice settings, a free-text note, reboot and shutdown |
| Videos | 29 | Library, titles, streaming, posters, subtitles, progress, episode management, settings, artwork refresh |
| Guest portal | 4 | Code verification, "am I authorised", a device's own address, and a tunnel-aware admin authorisation |
| **Total** | **81** | |

## Examples

Sign in:

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

Create a guest access code (the password is asked again):

```http
POST /api/portal/codes
{"label": "Workshop attendees", "code": "", "password": "••••••••"}

200 {"code": {"id": "…", "label": "Workshop attendees", "code": "7305", "uses": 0, "last_used": 0}}
```

A user account calling an administrator route:

```http
GET /api/clients
Authorization: Bearer <a user's token>

403 {"error": "This account cannot do that."}
```
