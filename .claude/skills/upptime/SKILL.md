---
name: upptime
description: >
  Expert guide for setting up, configuring, and managing Upptime — the GitHub-powered
  open-source uptime monitor and status page. Use this skill whenever the user wants to:
  set up a status page, configure .upptimerc.yml, add or modify monitored endpoints/sites,
  set up notifications (Slack, Discord, Telegram, email, SMS, webhooks), add custom domains,
  schedule maintenance windows, create status badges, troubleshoot GitHub Actions workflows,
  customize the status website appearance, or ask anything about Upptime configuration.
  Trigger even if the user just says "add a site to monitor", "my status page", "uptime check",
  "GitHub Actions monitor", or "how do I get notified when my site goes down".
---

# Upptime Skill

Upptime is a **serverless** uptime monitor and status page powered entirely by GitHub:
- **GitHub Actions** → checks endpoints every 5 minutes
- **GitHub Issues** → auto-created on downtime, auto-closed on recovery
- **GitHub Pages** → hosts the public status website

The main configuration lives in `.upptimerc.yml` at the repository root.

---

## Quick Reference: .upptimerc.yml Structure

```yaml
owner: your-github-username   # GitHub org or user owning this repo
repo: your-repo-name          # This repository's name
user-agent: your-github-username  # Required for GitHub API

sites:
  - name: My Service
    url: https://example.com

status-website:
  name: My Status Page
  cname: status.example.com   # Custom domain (or use baseUrl for GitHub Pages)
  # baseUrl: /your-repo-name  # Use this if no custom domain
```

---

## Sites / Endpoints Configuration

Each item under `sites:` defines one monitored endpoint:

```yaml
sites:
  # Basic HTTP check
  - name: My Website
    url: https://example.com

  # POST request with body
  - name: API Endpoint
    url: https://api.example.com/health
    method: POST
    body: '{"ping": true}'
    headers:
      - "Authorization: Bearer TOKEN"
      - "Content-Type: application/json"

  # TCP port check
  - name: Database
    url: db.example.com
    check: tcp-ping
    port: 5432

  # ICMP ping
  - name: Server
    url: server.example.com
    check: icmp-ping

  # Custom options
  - name: Slow API
    url: https://slow.example.com
    maxResponseTime: 5000        # Mark "degraded" if over 5s (default: 30000ms)
    expectedStatusCodes: [200, 201]  # Accept these HTTP codes as "up"
    slug: slow-api               # Custom slug for URLs and issue names
    assignees: [username1]       # Assign downtime issues to specific users
    ipv6: true                   # Force IPv6 DNS

  # Content-based checks (use sparingly — "dangerous" flags)
  - name: Check Page Content
    url: https://example.com
    __dangerous__body_down: "Service Unavailable"       # Down if text found
    __dangerous__body_down_if_text_missing: "Welcome"   # Down if text missing

  # Globalping (check from multiple global locations)
  - name: Global Check
    url: https://example.com
    type: globalping
    location: "North America, Europe"    # Continent, city, country, ASN, or ISP
```

---

## Status Website Customization

```yaml
status-website:
  name: My Status Page
  theme: dark          # Options: light, dark, night, ocean
  themeUrl: https://example.com/custom-theme.css  # Custom CSS (overrides theme)
  cname: status.example.com      # Custom domain
  # baseUrl: /repo-name          # GitHub Pages path (use instead of cname)
  logoUrl: https://example.com/logo.svg
  favicon: https://example.com/favicon.png
  introTitle: "**My Company** Status"     # Supports Markdown
  introMessage: "Real-time status of our services."
  navbar:
    - title: Home
      href: https://example.com
    - title: GitHub
      href: https://github.com/$OWNER/$REPO
  customHeadHtml: |
    <meta name="author" content="My Company">
  customBodyHtml: ""
  customFootHtml: "<p>Custom footer</p>"
  metaTags:
    - name: description
      content: "Service status page"
```

---

## Notifications Setup

**Important:** All notification credentials must be added as **GitHub Repository Secrets** (Settings → Secrets and variables → Actions → **Secrets**, NOT Variables).

Enable a provider by adding the matching secret variables:

### Slack
```
NOTIFICATION_SLACK=true
NOTIFICATION_SLACK_WEBHOOK=https://hooks.slack.com/services/XXX/YYY/ZZZ
```

### Discord
```
NOTIFICATION_DISCORD=true
NOTIFICATION_DISCORD_WEBHOOK=https://discord.com/api/webhooks/XXX/YYY
```

### Telegram
```
NOTIFICATION_TELEGRAM=true
NOTIFICATION_TELEGRAM_BOT_KEY=123456:ABC-DEF
NOTIFICATION_TELEGRAM_CHAT_ID=-1001234567890
```

### Microsoft Teams
```
NOTIFICATION_TEAMS=true
NOTIFICATION_TEAMS_WEBHOOK=https://xxx.webhook.office.com/webhookb2/...
```

### Email (SMTP)
```
NOTIFICATION_EMAIL=true
NOTIFICATION_EMAIL_FROM=alerts@example.com
NOTIFICATION_EMAIL_TO=team@example.com
NOTIFICATION_EMAIL_SMTP=smtp://user:pass@smtp.example.com:587
```

### Custom Webhook
```
NOTIFICATION_CUSTOM_WEBHOOK=true
NOTIFICATION_CUSTOM_WEBHOOK_URL=https://your-endpoint.com/hook
NOTIFICATION_CUSTOM_WEBHOOK_HEADERS={"Authorization": "Bearer token"}
NOTIFICATION_CUSTOM_WEBHOOK_BODY={"text": "$SITE_NAME is $STATUS"}
NOTIFICATION_CUSTOM_WEBHOOK_METHOD=POST
```

### Custom Message Variables
Use these in webhook body/messages:
- `$SITE_NAME` — site display name
- `$SITE_URL` — site URL
- `$STATUS` — "down" or "degraded"
- `$EMOJI` — status emoji

### Multiple Notification Strategies
When multiple configs of the same type exist:
```
NOTIFICATION_SLACK_STRATEGY=roundrobin  # Options: roundrobin (default), fallback, no-fallback
```

---

## Scheduled Maintenance Windows

Create a GitHub Issue in the repository with:
1. Apply the `maintenance` label
2. Add this HTML comment in the issue body:

```
<!--
start: 2024-03-15T10:00:00+08:00
end: 2024-03-15T12:00:00+08:00
expectedDown: service-slug-1, service-slug-2
expectedDegraded: api-slug
-->
```

- `start` / `end`: ISO 8601 datetime with timezone offset
- `expectedDown`: comma-separated slugs of sites that will be down (prevents false alerts)
- `expectedDegraded`: sites that will be slow/degraded
- Upptime **automatically closes** the issue when the window ends

---

## Status Badges

Add to any README or webpage (replace `OWNER` and `REPO`):

```markdown
<!-- Overall status -->
![Overall Status](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/OWNER/REPO/master/api/overall/uptime.json)

<!-- Per-site uptime (use the site's slug) -->
![My Site Uptime](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/OWNER/REPO/master/api/my-site/uptime.json)

<!-- Response time badge -->
![Response Time](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/OWNER/REPO/master/api/my-site/response-time.json)
```

The slug is derived from the site name (lowercased, spaces → hyphens) unless overridden with `slug:`.

---

## Workflow Scheduling

Override default cron schedules in `.upptimerc.yml`:

```yaml
workflowSchedule:
  uptime: "*/5 * * * *"         # Uptime check (default: every 5 min)
  responseTime: "0 23 * * *"   # Record response times (default: 11pm daily)
  graphs: "0 0 * * *"          # Generate graphs (default: midnight daily)
  summary: "0 0 * * *"         # Update summary (default: midnight daily)
  staticSite: "0 0 * * 0"      # Rebuild static site (default: weekly)
  updateTemplate: "0 0 * * 0"  # Self-update (default: weekly)
```

---

## Self-Hosted Runners

```yaml
runner: self-hosted                    # Single label
runner: [self-hosted, linux, x64]      # Multiple labels
```

---

## Commit Message Customization

```yaml
commitMessages:
  readmeContent: "ci: update readme with latest status [skip ci]"
  summaryJson: "ci: update summary JSON [skip ci]"
  statusChange: "ci: $SITE_NAME is $STATUS"
  graphsUpdate: "ci: update graphs [skip ci]"
  commitAuthorName: "GitHub Actions Bot"
  commitAuthorEmail: "actions@github.com"
```

---

## Internationalization (i18n)

```yaml
i18n:
  activeIncidents: "Active incidents"
  allSystemsOperational: "All systems operational"
  degradedPerformance: "Degraded performance"
  # See Upptime docs for full list of i18n keys
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Workflows not running | Go to Actions tab → enable workflows |
| 403 on GitHub API | Check `GH_PAT` secret has `actions`, `contents`, `issues`, `workflows` scopes |
| Status page not updating | Trigger "Setup CI" workflow manually from Actions tab |
| False downtime alerts | Add `expectedDown` to a maintenance issue, or check `expectedStatusCodes` config |
| SSL errors | Use `__dangerous__disable_verify_peer: true` (not recommended for production) |
| Site shows wrong status | Check the `api/<slug>/uptime.json` file in the repo |
| Build minutes exceeded | Switch to a public repo or reduce check frequency with `workflowSchedule.uptime` |

---

## Common Tasks

### Add a new site to monitor
Add an entry under `sites:` in `.upptimerc.yml` and push. The next uptime check (within 5 min) will include it.

### Remove a site
Delete its entry from `sites:` in `.upptimerc.yml`. Manually delete its `api/<slug>/` folder and history entries if you want to clean up.

### Change check frequency
Modify `workflowSchedule.uptime` cron expression. Remember: ~3,000 GitHub Actions minutes/month at 5-minute intervals for a free public repo.

### Force immediate re-check
Go to Actions → "Uptime CI" → "Run workflow".

### Update Upptime to latest version
Go to Actions → "Update Template CI" → "Run workflow", or wait for the weekly auto-update.

### Set up custom domain
1. Set `cname: yourdomain.com` in `.upptimerc.yml`
2. Add a CNAME DNS record pointing to `your-username.github.io`
3. Enable HTTPS in GitHub Pages settings

---

## Key Files in the Repository

```
.upptimerc.yml          ← Main configuration (edit this)
README.md               ← Auto-generated status summary (don't edit)
history/                ← Incident records (auto-managed)
api/                    ← JSON data for each endpoint (auto-managed)
graphs/                 ← Response time graphs (auto-managed)
.github/workflows/      ← GitHub Actions (auto-managed, don't edit directly)
```

> ⚠️ **Never edit files in `.github/workflows/` directly** — they are regenerated from `.upptimerc.yml` on every template update. All customization goes in `.upptimerc.yml`.
