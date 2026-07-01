# Account Health Agent

An AI-powered account intelligence tool that lets account managers ask natural language questions about their customer portfolio and get real-time churn risk analysis, renewal alerts, and recommended next actions — powered by the Claude AI API.

🔗 **[Live Demo](https://necker44.github.io/account-health-agent-v2/)**

---

## What it does

Instead of manually pulling CRM reports, you type a question like:

> *"Which accounts haven't been contacted in 60+ days and have renewals coming up?"*

The agent analyzes your full account portfolio, applies a weighted churn risk scoring model, and returns a prioritized list of accounts with specific recommended actions — all in seconds.

![Setup Screen](docs/setup-screen.png)

---

## Features

- **Natural language queries** — ask anything about your territory in plain English
- **AI-powered reasoning** — Claude reads the full account dataset, applies the scoring model, and explains its thinking
- **Churn risk scoring** — weighted model based on renewal proximity, contact gap, and product depth
- **Databricks integration** — architecture built to query live Delta Tables via the Databricks SQL Statement API
- **Zero dependencies** — single HTML file, runs in any browser with no installation

---

## Tech stack

| Layer | Technology |
|---|---|
| AI agent | [Claude API](https://anthropic.com) (`claude-sonnet-4-6`) |
| Data warehouse | [Databricks](https://databricks.com) Delta Lake |
| Data query | Databricks SQL Statement API |
| Frontend | Vanilla HTML / CSS / JS (single file, no framework) |
| Hosting | GitHub Pages |

---

## Churn risk scoring model

The agent uses a weighted scoring model to assess account health across three signal categories:

| Signal | Condition | Points |
|---|---|---|
| Renewal proximity | Within 30 days | +40 |
| Renewal proximity | Within 31–90 days | +20 |
| Contact gap | No contact in 90+ days | +35 |
| Contact gap | No contact in 60–89 days | +20 |
| Contact gap | No contact in 45–59 days | +10 |
| Product depth | Only 1 product | +15 |
| Contract length | 12-month contract | +10 |

**Risk tiers:** High (55+) · Medium (30–54) · Low (< 30)

---

## Architecture

```
┌─────────────────────────────────────────┐
│           Browser (GitHub Pages)         │
│                                          │
│   User question                          │
│        ↓                                 │
│   Account data (mock or Databricks)      │
│        ↓                                 │
│   Claude API call (Anthropic)            │
│        ↓                                 │
│   Structured JSON response               │
│        ↓                                 │
│   Rendered account cards + reasoning     │
└─────────────────────────────────────────┘
         ↑
         │  SQL Statement API
         │  (Databricks SQL Warehouse)
         │
┌─────────────────────────────────────────┐
│         Databricks Delta Lake            │
│                                          │
│   accounts table                         │
│   contracts table                        │
│   interactions table                     │
│   devices table                          │
└─────────────────────────────────────────┘
```

### Databricks connectivity — CORS constraint

The app is architected to connect directly to Databricks via the SQL Statement API, passing a Personal Access Token (PAT) and HTTP path from the browser connection drawer. In local development this works as expected.

When deployed to a public host like GitHub Pages, browsers enforce **CORS (Cross-Origin Resource Sharing)** policy — Databricks does not include the necessary headers to allow direct browser-to-API calls from external domains, which causes the connection to fail.

**Production solution:** Route Databricks calls through a lightweight backend proxy (Node.js/Express, FastAPI, or AWS Lambda) that sits between the browser and Databricks. The browser calls the proxy on the same domain, the proxy forwards the SQL request to Databricks with the PAT in the server-side header, and returns the result. CORS is never triggered because the request originates from a server, not a browser.

```
Browser → Backend Proxy → Databricks SQL Warehouse → Delta Tables
```

This is a standard pattern for securing data warehouse credentials in client-facing applications and would be the natural next step for a production deployment of this tool.

---

## Running locally (full Databricks connectivity)

To use the app with live Databricks data:

1. Download `index.html` and open it directly in your browser (not via a web server)
2. Enter your Claude API key on the setup screen
3. Click **⚡ Connect Databricks** and enter:
   - **Workspace URL** — your Databricks address bar URL
   - **Personal Access Token** — Settings → Developer → Access tokens → Generate new token
   - **HTTP Path** — SQL Warehouses → your warehouse → Connection details → HTTP path
   - **Schema** — `test_systems` (or `main.test_systems` with Unity Catalog)

When running as a local file (`file://`) the browser's CORS policy does not apply, so the Databricks connection works without a proxy.

---

## Databricks Delta table setup

Run `databricks/setup_tables.py` in a Databricks notebook to create the four Delta tables:

```sql
accounts      -- Primary table queried by the agent
contracts     -- Individual product contracts per account
interactions  -- Customer contact history log
devices       -- Hardware devices under management
```

The agent queries the `accounts` table directly. Column names must match:

```sql
account_id, account_name, industry, employees, mrr, region,
products, renewal_days, days_since_contact, contract_months
```

---

## Example questions

- *"Which accounts are at churn risk?"*
- *"Who hasn't been contacted in 60+ days?"*
- *"Show me renewals due in the next 90 days"*
- *"What's my total revenue at risk?"*
- *"Which single-product accounts should I upsell?"*
- *"Which healthcare accounts have upcoming renewals?"*
- *"Who should I call first thing Monday?"*

---

## SQL queries

The `sql/` folder contains standalone analytical queries that mirror the agent's logic:

| File | Description |
|---|---|
| `churn_risk_scores.sql` | Full churn scoring model for all accounts |
| `arr_by_risk_tier.sql` | ARR segmented by High / Medium / Low risk |
| `renewal_pipeline.sql` | Renewal calendar + priority outreach list |

---

## Project structure

```
account-health-agent-v2/
├── index.html                  # Full standalone app
├── databricks/
│   └── setup_tables.py         # Databricks notebook — creates Delta tables
├── sql/
│   ├── churn_risk_scores.sql
│   ├── arr_by_risk_tier.sql
│   └── renewal_pipeline.sql
├── .nojekyll                   # Disables Jekyll processing on GitHub Pages
└── README.md
```

---

## Security

- Claude API key and Databricks PAT are entered at runtime and held in browser memory only
- No credentials are stored, logged, or committed to this repository
- The `.nojekyll` file and lack of a backend mean there is no server-side code handling credentials
- For production use, credentials should be managed server-side via a backend proxy

---

## Background

This project demonstrates an agentic AI workflow applied to a real sales operations problem: account managers spend significant time manually pulling CRM reports to identify at-risk accounts. This tool replaces that workflow with a natural language interface backed by a structured data warehouse.

**Skills demonstrated:**
- Agentic AI application design (Claude API with structured JSON output and chain-of-thought reasoning)
- Databricks Delta Lake table design and SQL Warehouse integration architecture
- Churn risk modeling (multi-signal weighted scoring)
- Full-stack implementation (async API calls, dynamic UI rendering, error handling)
- Sales operations domain knowledge (renewal management, customer success signals)
- Understanding of browser security constraints (CORS) and production architecture patterns

---

## Author

Built by Nick — Account Manager transitioning into Sales Operations & Data Analytics

[GitHub](https://github.com/necker44) · [LinkedIn]www.linkedin.com/in/nickecker44
