# Product Requirements Document (PRD): US Options Management & Analytics Platform

## 1. Product Overview
* **Product Name:** Merlin (Working Title)
* **Objective:** Build a web-based options portfolio management, screening, and strategy analytics platform for the US stock market, modeled after the Brazilian *Eagle System*.
* **Data Source:** All real-time market quotes, option chains, Greeks, account balances, and open positions must be fetched directly from the user's **tastytrade** account via the official tastytrade REST and WebSocket (DXLink) APIs.
* **Target User:** Active US options retail traders focusing on income strategies (Covered Calls, Cash-Secured Puts), directional spreads, and multi-leg volatility structures.

---

## 2. Recommended Tech Stack (Optimized for AI Generation)
* **Frontend:** Next.js (App Router), React, TypeScript, Tailwind CSS, Shadcn/UI (for dense financial tables, modals, and clean dashboards), Recharts or Lightweight Charts (for payoff diagrams).
* **Backend / API Layer:** Next.js API Routes or Python (FastAPI) microservice to securely handle tastytrade authentication tokens, DXLink streamer connections, and Black-Scholes calculations.
* **Database:** PostgreSQL (via Prisma or Supabase) or SQLite (for local single-user deployment) to store user alerts, custom strategy tags, and historical roll tracking.
* **Market Data & Broker Integration:** `tastytrade` official REST API + `dxlink` WebSocket protocol for live quotes/Greeks.

---

## 3. Core Functional Requirements

### Module 1: Authentication & Broker Sync (tastytrade API)
* **Session Management:** Support tastytrade API authentication (OAuth2 or Session Token via `/sessions` endpoint) with secure encrypted storage and automatic token refresh.
* **Account Sync:** Fetch and display account net liquidating value, buying power (equity & options), maintenance excess, and cash balance.
* **Position Ingestion:** Automatically pull all open equity and option positions via `/accounts/{account_number}/positions` and group option legs into logical multi-leg structures (e.g., matching short calls with underlying shares as a Covered Call).

### Module 2: Options Screener ("Buscador de Opções")
* **Chain Retrieval:** Fetch full option chains for any US ticker via `/option-chains/{symbol}/nested`.
* **Live Market Data:** Stream real-time Bid, Ask, Mid, Last, Open Interest, Volume, Implied Volatility (IV), and Greeks (Δ, Γ, Θ, ν) via DXLink.
* **Filtering Engine:** Allow users to filter contracts by:
  * Option Type (Calls / Puts)
  * Expiration Date / Days to Expiration (DTE) range
  * Delta range (e.g., 0.15 to 0.35)
  * Minimum Open Interest / Volume (liquidity filter)
  * Annualized Return on Risk / Return on Capital (Yield)
  * Moneyness (ITM, ATM, OTM) and distance from current underlying price (%)

### Module 3: Position Management & Rolling Assistant ("Gestão e Rolagem")
* **Strategy Grouping:** Display active positions grouped by underlying asset and strategy type (e.g., Covered Call, Wheel, Bull Put Spread, Iron Condor).
* **Live P&L & Greeks:** Calculate real-time unrealized P&L ($ and %), net position Delta, and daily Theta decay.
* **Smart Roll Calculator:**
  * Provide a "Simulate Roll" action on any short option or spread.
  * Fetch later expiration chains for the same underlying.
  * Calculate **Net Credit / Net Debit** of closing the current leg(s) and opening the new leg(s).
  * Display new break-even price, new annualized yield, and change in strike price.

### Module 4: Multi-Leg Strategy Builder ("Estruturas Avançadas")
* **Payoff Diagram:** Interactive P&L expiration graph showing profit/loss zones, break-even points, Max Profit, Max Loss, and Probability of Profit (POP).
* **Pre-built Templates:** One-click setup for Covered Call, Cash-Secured Put, Vertical Spreads (Bull/Bear), Iron Condor, Strangle, and Calendar Spread.
* **What-If Simulator:** Sliders to simulate changes in underlying stock price, Implied Volatility (± IV%), and Days to Expiration (time decay curve).

### Module 5: Market & Volatility Analytics ("Análise de Mercado")
* **Volatility Metrics:** Fetch and display IV Rank (IVR), IV Percentile, 30-day Historical Volatility (HV), and earnings dates via tastytrade's market metrics endpoint (`/market-metrics`).
* **Dividend & Earnings Calendar:** Flag upcoming ex-dividend dates (critical for early assignment risk on short calls) and earnings announcements for all tickers in the user's portfolio or watchlist.

### Module 6: Smart Alerts ("Alertas Inteligentes")
* **Trigger Conditions:** Users can set custom alerts based on:
  * Underlying stock price crossing a threshold.
  * Option contract reaching a target profit % (e.g., "Alert when short put reaches 50% max profit").
  * Delta threshold breach (e.g., "Alert when short call Delta exceeds 0.60").
  * DTE countdown (e.g., "Alert at 21 DTE to manage/roll").
* **Notification Channels:** In-app toast notifications, Email (Resend/SMTP), and Telegram Bot webhook integration.

---

## 4. Non-Functional Requirements & Constraints
* **Rate Limiting & Caching:** Cache static data (option chain expirations, instrument metadata, earnings dates) to respect tastytrade API rate limits, while keeping quotes and Greeks real-time via WebSockets.
* **Read-Only Default:** Initially build order execution as **dry-run / order ticket preview** (using tastytrade's `/accounts/{account_number}/orders/dry-run` endpoint) before enabling live order routing, ensuring safety during development.
* **UI/UX Design:** Dark-mode-first financial terminal aesthetic, high data density, color-coded P&L and Greeks, and responsive tables with sorting/pinning.

---

## 5. Phased Implementation Roadmap (For AI Agent Execution)
* **Phase 1: Project Setup & tastytrade Auth.** Scaffold project, configure environment variables (`TASTYTRADE_USERNAME`, `TASTYTRADE_PASSWORD` or OAuth credentials), implement login/session service, and display basic account balances.
* **Phase 2: Portfolio & Position Grouping.** Fetch open positions, build the strategy-grouping algorithm, and render the main Portfolio Dashboard.
* **Phase 3: DXLink Streamer & Option Chain Screener.** Integrate live WebSocket quotes/Greeks and build the filterable Options Screener table.
* **Phase 4: Strategy Builder & Roll Assistant.** Implement the payoff chart component, Black-Scholes pricing helpers, and the interactive Roll Simulator.
* **Phase 5: Market Metrics & Alerts Engine.** Connect `/market-metrics` for IVR/Earnings and build the background worker for Telegram/Email alerts.

---

### How to use this PRD with an AI Coding Agent
1. Save this text as `PRD.md` in the root folder of a new project.
2. Prompt your AI agent with: *"Read PRD.md. Initialize the project using the recommended tech stack and start executing Phase 1. Ask me for any API keys or environment preferences before proceeding."*
