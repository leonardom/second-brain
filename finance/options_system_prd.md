# Product Requirement Document (PRD)

## Project Name: Automated Options Strategy & Selection Engine (AOSSE)
**Author:** AI Technical Collaborator  
**Date:** September 2026  
**Status:** Draft / Specification  

---

## 1. Executive Summary & Core Objective
The objective of this project is to build a custom, Python-based algorithmic options scanner and strategy selector. The system will ingest underlying stock information alongside real-time/historical option chains scraped or extracted from **opcoes.net**. 

By running data through a quantitative evaluation engine, the platform will automatically filter out low-probability trades and output the absolute **best stock option contracts** paired with the **most mathematically optimized options strategy** (e.g., Credit Spreads, Covered Calls, Debit Spreads, or Iron Condors) tailored to current market conditions.

---

## 2. Risk & Portfolio Management Disclaimer
*   **Leverage Warning:** Options trading involves high volatility and massive potential leverage. Software bugs, API changes, or logical errors can cause severe, accelerated capital loss.
*   **Contextual Constraints:** The system must be built and evaluated purely as a data-filtering and decision-support mechanism within a broader, diversified portfolio context. It should never execute trades autonomously without manual, human-in-the-loop validation during the pilot phase.
*   **Paper Trading Mandatory Requirement:** The system must go through a mandatory minimum of **90 days of paper trading** (simulated execution) to log and audit edge cases, API parsing failures, and historical model deviations before transitioning to live capital.

---

## 3. Scope & System Architecture

### 3.1 In-Scope Requirements
*   Automated extraction and structural parsing of underlying stock ticker data and structured option chains from **opcoes.net**.
*   Mathematical calculation of Implied Volatility (IV) Rank, IV Percentile, and foundational Option Greeks (Delta, Theta).
*   A conditional strategy routing engine mapping current stock trends and volatility environments to optimized risk/reward structures.
*   A clean terminal or local UI dashboard outputting categorized trade recommendations containing explicit strike prices, expiration dates, Maximum Profit, Maximum Loss, and Break-Even thresholds.

### 3.2 Out-of-Scope Requirements (Phase 1)
*   Direct API order execution or auto-routing to brokerages.
*   Portfolio-wide automated rebalancing algorithms.
*   Handling of complex multi-leg synthetic equity instruments (e.g., custom ratios, rolling calendars).

---

## 4. Functional Specifications & Data Pipeline

```
[ opcoes.net Data Extraction ] ──► [ Quant Engine: IV Rank & Greeks ]
                                                 │
                                                 ▼
[ User UI Dashboard Output ]   ◄── [ Strategy Selector Matrix ]
```

### 4.1 Phase 1: Data Extraction Pipeline (`opcoes.net`)
*   **Source Data:** Custom Python scrapers/parsers built utilizing `requests`, `BeautifulSoup`, or `Playwright` to extract target data fields from **opcoes.net** (focusing on Brazilian options/B3 equity options market if applicable).
*   **Required Ingestion Payload (Per Ticker):**
    *   Underlying asset spot price.
    *   Option Type (Call vs. Put).
    *   Strike Price ($K$) and Expiration Date ($T$).
    *   Current Bid / Ask spread and Last Traded Price.
    *   Implied Volatility (IV) per contract.
    *   Open Interest and Volume (for liquidity health tracking).

### 4.2 Phase 2: Quantitative Engine & Indicator Computation
The engine processes raw unstructured arrays into key decision metrics:
*   **IV Rank Calculation:** Tracks current IV relative to its 52-week absolute high and low points.
    $$	ext{IV Rank} = rac{	ext{Current IV} - 	ext{52W Low IV}}{	ext{52W High IV} - 	ext{52W Low IV}} 	imes 100$$
*   **Liquidity Filter:** Automatically drops options with bid-ask spreads exceeding a user-defined threshold percentage of the mid-price to protect against high slippage.

### 4.3 Phase 3: Strategy Selection Logic Matrix
The algorithm routes ticker inputs based on a rigid conditional state matrix:

| Stock Trend Direction | Volatility Context (IV Rank) | Recommended Strategy | Primary Mechanics |
| :--- | :--- | :--- | :--- |
| **Bullish** | High (IV Rank > 60) | **Bull Put Credit Spread** | Sells expensive downside premium; accelerates on positive time decay ($\Theta$). |
| **Bullish** | Low (IV Rank < 40) | **Bull Call Debit Spread** | Buys cheap upside calls; caps maximum risk to net debit paid. |
| **Bearish** | High (IV Rank > 60) | **Bear Call Credit Spread** | Sells expensive upside premium above directional resistance. |
| **Bearish** | Low (IV Rank < 40) | **Long Put / Bear Put Spread**| Purchases inexpensive downside puts to capture rapid negative moves. |
| **Neutral** | Extremely High (> 70) | **Iron Condor** | Simultaneously sells out-of-the-money Call & Put spreads; extracts premium from a sideways market. |

### 4.4 Phase 4: Output & Reporting Specifications
The output console or local web app (e.g., Streamlit) must explicitly present:
1.  **Ticker Name & Underlying Price**
2.  **Selected Strategy Struct** (e.g., *Buy 100 Strike Call / Sell 105 Strike Call*)
3.  **Risk Metrics:** Maximum Profit, Maximum Loss, Net Debit/Credit, Risk-to-Reward Ratio, and Theoretical Probability of Profit (PoP) based on delta distributions.

---

## 5. Non-Functional & Technical Requirements
*   **Language & Stack:** Python 3.10+, Pandas, NumPy, SciPy (for Black-Scholes verifications), and Streamlit or Rich for terminal UI layout formatting.
*   **Robustness & Exception Handling:** The data pipeline must handle missing data points, halted options trading brackets, and structural layout adjustments on `opcoes.net` without crashing the core execution script.
*   **Rate-Limiting Compliance:** Web extraction logic must include randomized sleep intervals ($2	ext{s} - 5	ext{s}$) to remain respectful of target server resources and prevent automated IP blocking.

---

## 6. Success Metrics & Validation Checklist
*   [ ] **Parsing Accuracy:** 100% precision validating extracted numerical fields (Strike, Expiration, Spot Price) against raw webpage values.
*   [ ] **Latency Rule:** Strategy filtering matrix execution takes under 1.5 seconds per complete option chain once local data tables are hydrated.
*   [ ] **Simulation Safety:** System correctly generates 0-dollar simulated tracking statements during its mandatory 90-day sandbox trial.
