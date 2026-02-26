# ZeptoClaw Community Skills

> Community-contributed skills for [ZeptoClaw](https://github.com/qhkm/zeptoclaw) — the ultra-lightweight AI agent runtime.

## What is a Skill?

A skill is a markdown file (`SKILL.md`) that teaches ZeptoClaw how to perform a specific task — using tools, APIs, or workflows. Skills are injected into the agent's context when needed.

```
zeptoclaw skills search "web scraping"   # discover skills
zeptoclaw skills install --github qhkm/zeptoclaw-skills   # install all
```

## Skills in This Repo

| Skill | Description | Tags | Author |
|-------|-------------|------|--------|
| [airtable](./airtable/) | Read, create, update, and delete Airtable records via the REST API | airtable, database, productivity | ZeptoClaw |
| [billplz](./billplz/) | Create bills and check payments via Billplz — Malaysia's payment gateway | billplz, payments, sea | ZeptoClaw |
| [clawpocket](./clawpocket/) | Publish trade signals to ClawPocket marketplace | crypto, defi | [@aimaneth](https://github.com/aimaneth) |
| [cloudflare-ops](./cloudflare-ops/) | Manage Cloudflare DNS records, Workers, KV, and R2 via the API v4 | cloudflare, dns, devops | ZeptoClaw |
| [discord-webhook](./discord-webhook/) | Send messages and rich embeds to Discord channels via webhooks | discord, messaging, notifications | ZeptoClaw |
| [docker-deploy](./docker-deploy/) | Deploy, update, and manage Docker Compose services in production | docker, devops | ZeptoClaw |
| [git-release](./git-release/) | Automate semver releases — bump version, generate changelog, tag, and publish GitHub Release | git, devops, semver | ZeptoClaw |
| [google-calendar](./google-calendar/) | List, create, update, and delete Google Calendar events via Calendar API v3 | google, calendar, productivity | ZeptoClaw |
| [grab-merchant](./grab-merchant/) | Manage GrabFood/GrabMart orders, menus, and store hours via Merchant API | grab, food-delivery, sea | ZeptoClaw |
| [lazada](./lazada/) | Lazada Seller Center API — manage products, orders, and inventory across SEA | lazada, ecommerce, sea | ZeptoClaw |
| [linear](./linear/) | Manage Linear issues, projects, and teams via the GraphQL API | linear, issues, project-management | ZeptoClaw |
| [make-invoice](./make-invoice/) | Generate professional HTML/PDF invoices from order data and email them | invoice, billing, pdf | ZeptoClaw |
| [notion](./notion/) | Read, create, and update Notion pages and databases via the official API | notion, productivity | ZeptoClaw |
| [postgres](./postgres/) | PostgreSQL queries, schema management, backups, and performance tuning | database, sql | ZeptoClaw |
| [send-email](./send-email/) | Send transactional emails via Resend — order confirmations, alerts, reports | email, notifications | ZeptoClaw |
| [sentry](./sentry/) | Monitor and resolve Sentry issues, view project stats, and manage errors | sentry, monitoring, devops | ZeptoClaw |
| [shopee](./shopee/) | Malaysian e-commerce helper for Shopee seller operations | shopee, ecommerce, sea | ZeptoClaw |
| [slack-msg](./slack-msg/) | Post messages, list channels, and reply to threads via Slack Web API | slack, messaging, communication | ZeptoClaw |
| [supabase](./supabase/) | Query tables, insert rows, and manage auth users via Supabase PostgREST | supabase, database, backend | ZeptoClaw |
| [summarize-url](./summarize-url/) | Fetch a URL and extract clean text content for summarization or analysis | web, scraping, research | ZeptoClaw |
| [telegram-notify](./telegram-notify/) | Send Telegram notifications and alerts via Bot API | telegram, notifications | ZeptoClaw |
| [tiktok-shop](./tiktok-shop/) | Manage TikTok Shop products, orders, and inventory via Open API v2 | tiktok, ecommerce, sea | ZeptoClaw |
| [todoist](./todoist/) | Manage Todoist tasks and projects via the REST API v2 | todoist, tasks, productivity | ZeptoClaw |
| [tokopedia](./tokopedia/) | Manage Tokopedia shop products, orders, and inventory via Seller API | tokopedia, ecommerce, sea | ZeptoClaw |
| [twilio-sms](./twilio-sms/) | Send and manage SMS messages via the Twilio API | twilio, sms, communication | ZeptoClaw |
| [vercel](./vercel/) | Manage Vercel deployments, projects, and domains via the REST API | vercel, deployment, devops | ZeptoClaw |
| [weather](./weather/) | Get current weather and forecasts without API keys | weather, utilities | ZeptoClaw |
| [whatsapp-blast](./whatsapp-blast/) | Send bulk WhatsApp messages via WhatsApp Cloud API for SEA businesses | whatsapp, sea, ecommerce | ZeptoClaw |
| [template](./template/) | Starter template for new skills | — | ZeptoClaw |

## Contributing a Skill

1. **Fork** this repo
2. **Copy** the `template/` directory and rename it to your skill name
3. **Edit** `SKILL.md` with your skill's instructions
4. **Open a PR** — include a brief description and at least one tested example

### Skill naming rules
- Lowercase, hyphens only (e.g. `my-skill`)
- No duplicates — check existing skills first
- Name should match the `name:` field in the YAML frontmatter

### What makes a good skill?
- Solves a real, repeatable task
- Has clear, copy-pasteable examples
- Declares its dependencies (`env_needed`, `requires`)
- Works without modifying ZeptoClaw's core code

### What we won't accept
- Skills that require proprietary, closed-source tooling with no alternatives
- Skills that automate harmful, illegal, or unethical actions
- Duplicate skills (refine an existing one instead)

## Skill Format

```markdown
---
name: my-skill
version: 1.0.0
description: One-line description of what this skill does.
author: Your Name
license: MIT
tags:
  - category
env_needed:
  - name: MY_API_KEY
    description: API key for the service
    required: true
metadata: {"zeptoclaw":{"emoji":"🛠️","requires":{"anyBins":["curl"]}}}
---

# My Skill

Explain what this skill does and when to use it.

## Usage

```bash
# Example command
```
```

> **Note:** Use `anyBins` (not `bins`) in the `requires` field — that's the key ZeptoClaw checks.

## License

All skills in this repo are MIT licensed unless the individual skill specifies otherwise.
