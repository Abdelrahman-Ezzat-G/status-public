# Abdelrahman Ezzat — Status Page

[![Status](https://img.shields.io/badge/status-operational-10b981)](https://status.abdelrahman-ezzat.com)
[![Uptime Check](https://github.com/Abdelrahman-Ezzat-G/status-public/actions/workflows/uptime.yml/badge.svg)](https://github.com/Abdelrahman-Ezzat-G/status-public/actions/workflows/uptime.yml)

Personal status page for [Abdelrahman Ezzat](https://abdelrahman-ezzat.com) — real-time uptime, response-time and **TLS certificate** monitoring for every personal & QuNeva service.

**Live Status Page:** [https://status.abdelrahman-ezzat.com](https://status.abdelrahman-ezzat.com)

---

## Monitored Services

| Service | URL | Notes |
|---------|-----|-------|
| Personal Portfolio | https://abdelrahman-ezzat.com | |
| Home Assistant | https://ha.abdelrahman-ezzat.com | 401/403 count as *up* (auth wall = alive) |
| QuNeva Platform | https://quneva.com | probes `/healthz` (bot-protection exempt) |
| QuNeva AI | https://ai.quneva.com | |
| QuNeva Status Page | https://status.quneva.com | |

All services are defined in **one file — [`services.json`](services.json)** — used by both the monitor workflow and the dashboard.

---

## How It Works

- **GitHub Actions** runs checks on a quarter-hourly cron (`uptime.yml`), driven entirely by `services.json`. GitHub schedules are best-effort, so real runs land every ~15-60 minutes; the page shows the observed cadence and warns if data goes stale
- Each check records HTTP status, response time and **TLS certificate days-to-expiry**
- Results are saved to `data/status.json`, `data/history.csv` and monthly `data/history-archive-YYYY-MM.csv`
- **GitHub Pages** hosts the dashboard (`index.html`) at the custom domain
- Outages automatically open a GitHub Issue labelled `incident` and close it on recovery
- Certificates expiring within **10 days** open an issue labelled `ssl-expiry` (auto-closed after renewal)
- Incidents are exported to `data/incidents.json` and an Atom feed at `data/incidents.xml`
- Optional email alerts fire on **state changes only** (new outage / full recovery / new cert warning)

---

## Adding a New Monitor

Add one entry to `services.json` — that's it. The workflow and dashboard both pick it up automatically:

```json
{
  "id": "myservice",
  "name": "My Service",
  "url": "https://example.com",
  "check_url": "https://example.com/healthz",
  "icon": "🚀",
  "extra_ok": [401],
  "group": "Personal"
}
```

- `check_url` (optional) — probe a different URL than the one displayed
- `extra_ok` (optional) — HTTP codes outside 200–399 that still count as *up*
- `slow_ms` (optional) — per-service "slow" threshold in ms (default 800); use for services with a naturally higher baseline so they aren't permanently flagged as degraded
- `group` (optional) — small badge shown on the service card

The history CSV is header-mapped, so adding/reordering services never corrupts old data.

---

## Email Alerts (optional)

Alerts fire only on **state changes**, never every run. Add repository secrets (**Settings → Secrets and variables → Actions**):

| Secret | Required | Default | Notes |
|--------|----------|---------|-------|
| `RESEND_API_KEY` | ✅ | — | API key from [Resend](https://resend.com) |
| `MAIL_FROM` | — | `AE Status <status@abdelrahman-ezzat.com>` | Must be a **verified domain** in Resend |
| `MAIL_TO` | — | `abdelrahman-ezzat@outlook.com` | Where alerts are delivered |

**Fallback — plain SMTP:** set `MAIL_USERNAME` + `MAIL_PASSWORD` (and optionally `SMTP_HOST` / `SMTP_PORT`, defaulting to `smtp-mail.outlook.com:587`). Resend is tried first when `RESEND_API_KEY` is present.

---

## Posting a Scheduled Maintenance Notice

Create a GitHub Issue and add the **`maintenance`** label. While the issue is open it appears as a banner on the status page. Close the issue to remove the banner.

---

## Repository Structure

```
status-public/
├── index.html                    ← Dashboard (hosted on GitHub Pages)
├── services.json                 ← Single source of truth for monitored services
├── 404.html                      ← Branded not-found page
├── CNAME                         ← Custom domain: status.abdelrahman-ezzat.com
├── manifest.webmanifest          ← PWA manifest (add to home screen)
├── favicon.svg / icon-*.png      ← Brand icons
├── og.png                        ← Social share image
├── robots.txt / sitemap.xml      ← SEO
├── data/
│   ├── status.json               ← Latest check results incl. SSL expiry (auto-updated)
│   ├── history.csv               ← Rolling recent history, last 2000 rows (auto-updated)
│   ├── history-archive-*.csv     ← Permanent monthly archive (append-only)
│   ├── archives.json             ← Manifest of available archives
│   ├── incidents.json            ← Recent incidents, last 30 days (auto-updated)
│   ├── incidents.xml             ← Atom/RSS feed of incidents (auto-updated)
│   └── maintenance.json          ← Open maintenance notices (auto-updated)
└── .github/workflows/
    └── uptime.yml                ← Monitor (quarter-hourly cron, best-effort)
```

---

## DNS Setup (where abdelrahman-ezzat.com is managed)

```
Type:   CNAME
Name:   status
Target: abdelrahman-ezzat-g.github.io
Proxy:  DNS Only (grey cloud — NOT orange, if using Cloudflare)
TTL:    Auto
```

Then in **Settings → Pages** set the custom domain to `status.abdelrahman-ezzat.com` and enable **Enforce HTTPS** once the certificate is issued.

---

## GitHub Actions Permissions Required

Go to **Settings → Actions → General**:
- Workflow permissions: **Read and write permissions** ✅

---

*© 2026 [Abdelrahman Ezzat](https://abdelrahman-ezzat.com)*
