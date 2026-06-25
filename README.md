# Paidt — Architecture

> An AI-powered Meta (Facebook/Instagram) Ads management SaaS. Paidt connects to a
> user's ad account and helps them plan, create, evaluate, and manage campaigns — with
> AI agents that can read the account, propose changes, and (after confirmation) execute
> them through the Graph API.

**Status:** In production · **Role:** Sole developer — architecture, implementation, and operations.
**Platform:** Approved through Meta's App Review (Facebook Login for Business).

> 📄 This repository documents the **architecture** of Paidt. The product source code is
> private; what's published here is the design and the engineering decisions behind it.
> Scale, for context: ~60 Postgres tables and ~90 Supabase Edge Functions in production.

---

## System map

Paidt is a broad product. The subsystems below each own a slice of the funnel, from
connecting an ad account to creating creatives, running AI agents, and billing.

| Subsystem | What it does |
|---|---|
| **Meta integration / OAuth** | Facebook Login for Business connect flow, ad-account & pixel selection, token lifecycle, and the data-deletion callback required by App Review. |
| **Creative pipeline** | Raw media → AI analysis (description, hook, transcript) → ad creatives. A creative maps directly onto Meta's `asset_feed_spec` placement rules and carousel cards, plus a predictive "pre-flight" score. |
| **AI agents** | A Meta-account agent that reads and acts on the account (see deep-dive), plus assistants for creative lab, landing-page editing, and context building. |
| **Competitor spy** | A scraping pipeline (APIFY + Microlink + Gemini) that ingests competitor ads with per-stage telemetry. |
| **Landing pages** | An in-house page builder (JSON document), immutable publishing, form capture, click-heatmap analytics, A/B, and Meta CAPI attribution. |
| **Billing** | Stripe checkout / portal / trial / idempotent webhook, per-plan spend quotas, a lifetime free tier, and a cancellation/retention flow. |
| **AI cost observability (FinOps)** | Per-call cost capture with price snapshots and a weekly drift monitor against an external pricing source (see deep-dive). |
| **Security & limits** | Per-kind and per-IP-hash rate limiting (LGPD-safe), a quota gate, and CSP violation reporting. |
| **Admin panel** | A separate app (~35 functions) for CAC, journey, drop-off, retention, financials, and per-subsystem health. |

---

## Deep dives

The two subsystems with the most interesting engineering are documented in depth:

| Document | Why it's worth reading |
|---|---|
| [AI agent that acts on Meta](docs/ai-agent-meta.md) | An agent with tool-use that **executes real changes** on a live ad account — built around a stage-then-confirm safety model, a full audit log, and bounded cost. |
| [AI cost observability (FinOps)](docs/ai-finops.md) | Every paid model/API call is costed and recorded with a **price snapshot**, plus a monitor that flags when prices drift from an external source. |

---

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | React + Vite |
| Compute | Supabase Edge Functions (Deno), ~90 in production |
| Data | Supabase Postgres (RLS on every table), `pg_cron` + `pg_net` for schedules |
| AI | Google Gemini (agents, analysis), OpenAI (images), Perplexity (search) |
| Ads platform | Meta Graph API `v25.0` (Facebook Login for Business) |
| Billing | Stripe (subscriptions, portal, webhooks) |
| Scraping | APIFY + Microlink |
| Observability | Sentry, mirrored into an in-app health panel |

---


