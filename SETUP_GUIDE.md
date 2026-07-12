# Setup Guide — abdelrahman-ezzat-status

One-time steps to take this repository live at **https://status.abdelrahman-ezzat.com**.

## 1. GitHub Pages

1. Go to **Settings → Pages**
2. Under *Build and deployment*, set **Source** to **GitHub Actions**
3. Under *Custom domain*, enter `status.abdelrahman-ezzat.com` and save
4. Wait for the DNS check to pass, then tick **Enforce HTTPS**

## 2. DNS record

At the DNS provider for `abdelrahman-ezzat.com`, add:

```
Type:   CNAME
Name:   status
Target: abdelrahman-ezzat-g.github.io
Proxy:  DNS Only (grey cloud — NOT orange, if using Cloudflare)
TTL:    Auto
```

> The grey cloud matters: GitHub Pages must see the CNAME directly to issue the TLS certificate.

## 3. Actions permissions

**Settings → Actions → General → Workflow permissions** → select **Read and write permissions** and save. The monitor needs this to commit `data/` updates and open/close incident issues.

## 4. Labels (optional but recommended)

Create these issue labels (**Issues → Labels**), used by the automation:

| Label | Purpose |
|-------|---------|
| `incident` | Auto-created on outage, auto-closed on recovery |
| `ssl-expiry` | Auto-created when a TLS certificate has ≤ 10 days left |
| `maintenance` | Add to any issue to show a maintenance banner on the page |

(GitHub auto-creates missing labels on first use, but pre-creating them lets you pick colors.)

## 5. Email alerts (optional)

Add repository secrets under **Settings → Secrets and variables → Actions**:

- `RESEND_API_KEY` — recommended; from [resend.com](https://resend.com) with `abdelrahman-ezzat.com` verified as a sending domain
- or `MAIL_USERNAME` + `MAIL_PASSWORD` for plain SMTP (Outlook by default)

No secrets → monitoring still works fully; you just get no emails (incidents still appear as GitHub Issues, and GitHub notifies you about new issues by default).

## 6. First run

Trigger the monitor once manually: **Actions → Uptime Monitor → Run workflow**. Within a minute:

- `data/status.json` gets real results (including SSL days-to-expiry)
- the first history row lands in `data/history.csv`
- the deploy workflow publishes the page

From then on it runs automatically every 10 minutes.

## Troubleshooting

- **Page shows "Monitoring warming up"** — the first uptime run hasn't completed yet.
- **Custom domain certificate pending** — flip the Cloudflare record to *DNS Only* (grey) and re-save the custom domain in Pages settings.
- **Workflow can't push** — re-check step 3 (write permissions).
- **A service shows down but is actually fine** — if it sits behind an auth wall or bot protection, give it a `check_url` health endpoint or add the expected code to `extra_ok` in `services.json`.
