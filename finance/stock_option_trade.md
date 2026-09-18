# 💻 The Python Monitoring System

To run this, you will need to install the required libraries:

```shell
pip install yfinance pandas pandas-ta
```

```python
import yfinance as yf
import pandas as pd
import pandas_ta as ta

def analyze_bull_put_setup(ticker_symbol):
    print(f"🔎 Scanning {ticker_symbol}...")
    
    # 1. Fetch Daily Historical Stock Data
    ticker = yf.Ticker(ticker_symbol)
    df = ticker.history(period="1y", interval="1d")
    
    if len(df) < 200:
        print("❌ Not enough historical data.")
        return

    # 2. Calculate Indicators using pandas_ta
    df['SMA_50'] = ta.sma(df['Close'], length=50)
    df['SMA_200'] = ta.sma(df['Close'], length=200)
    
    # Stochastic (14, 3, 3) -> Returns %K (fast) and %D (slow)
    stoch = ta.stoch(df['High'], df['Low'], df['Close'], k=14, d=3, smooth_k=3)
    df = pd.concat([df, stoch], axis=1)
    
    # Rename columns for easier access (exact string names depend on library version)
    k_col = [col for col in df.columns if 'STOCHk' in col][0]
    d_col = [col for col in df.columns if 'STOCHd' in col][0]

    # Get current (today's) and previous (yesterday's) data points
    current_row = df.iloc[-1]
    prev_row = df.iloc[-2]
    
    current_price = current_row['Close']
    sma50 = current_row['SMA_50']
    sma200 = current_row['SMA_200']
    
    k_today, d_today = current_row[k_col], current_row[d_col]
    k_yesterday, d_yesterday = prev_row[k_col], prev_row[d_col]

    # 3. Check Strategic Conditions
    is_bullish_trend = current_price > sma50 > sma200
    was_oversold = k_yesterday < 20 or d_yesterday < 20
    is_crossover = k_yesterday <= d_yesterday and k_today > d_today

    if is_bullish_trend and was_oversold and is_crossover:
        print(f"✅ TRIGGER DETECTED for {ticker_symbol} at ${current_price:.2f}!")
        print(f"   Stochastic Crossover in Oversold Zone (%K: {k_today:.1f} crossed above %D: {d_today:.1f})")
        
        # 4. Fetch Options Chain & Calculate Legs
        calculate_options_legs(ticker, current_price)
    else:
        print(f"❌ No clear setup for {ticker_symbol}. (Stoch K: {k_today:.1f}, D: {d_today:.1f})")

def calculate_options_legs(ticker_obj, current_price):
    # Target 30-45 Days to Expiration (DTE)
    expirations = ticker_obj.options
    if not expirations:
        print("   ⚠️ No options data available.")
        return
        
    # Pick the first expiration available after 30 days
    # (In production, parse string dates to select exactly 30-45 DTE)
    target_expiration = expirations[2] 
    print(f"   📅 Target Expiration: {target_expiration}")
    
    opt_chain = ticker_obj.option_chain(target_expiration)
    puts = opt_chain.puts
    
    # Filter for OTM Puts below the current price
    otm_puts = puts[puts['strike'] < current_price].copy()
    
    # Filter using basic Delta target or proxy (Yahoo Finance data often lacks live Greek columns,
    # so we approximate using strike proximity or calculate via implied volatility).
    # For a free programmatic proxy: target a short strike ~5% to 8% below current price.
    short_strike_target = current_price * 0.94
    
    # Find closest available strike for Short Put
    otm_puts['diff'] = (otm_puts['strike'] - short_strike_target).abs()
    short_put_row = otm_puts.loc[otm_puts['diff'].idxmin()]
    short_strike = short_put_row['strike']
    
    # Select Long Put (typically 5 dollars below the short put for a standard spread)
    long_strike = short_strike - 5
    
    print(f"   🚀 PROPOSED BULL PUT SPREAD LEGS:")
    print(f"      1️⃣ SELL (Short) Strike: ${short_strike} Put")
    print(f"      2️⃣ BUY  (Long)  Strike: ${long_strike} Put")
    print(f"      💰 Approximate Margin Required: ${5 * 100} per contract")

# Example Run
analyze_bull_put_setup("AAPL")
```

---

# 💻 Production Script: Tastytrade + SMA + Stochastic

To run this code, make sure you install the SDK:

```shell
pip install tastytrade pandas pandas-ta yfinance
```


```python
import yfinance as yf
import pandas as pd
import pandas_ta as ta
from datetime import datetime, timedelta
from tastytrade import Session, ProductionSession
from tastytrade.metrics import get_market_metrics
from tastytrade.instruments import OptionChain

# 1. Authenticate with Tastytrade
# Change to your real Tastytrade credentials. Production is required for real-time market data.
USERNAME = "your_tastytrade_username"
PASSWORD = "your_tastytrade_password"

try:
    session = ProductionSession(USERNAME, PASSWORD)
    print("🔒 Tastytrade Session Authenticated Successfully.")
except Exception as e:
    print(f"❌ Login Failed: {e}")
    exit()

def run_bull_put_scanner(ticker_symbol):
    print(f"\n🔎 Scanning {ticker_symbol}...")
    
    # ----------------------------------------------------
    # STEP 1: Fetch Technical Indicators via yfinance
    # ----------------------------------------------------
    stock = yf.Ticker(ticker_symbol)
    df = stock.history(period="1y", interval="1d")
    
    if len(df) < 200:
        print("❌ Not enough technical historical data.")
        return

    df['SMA_50'] = ta.sma(df['Close'], length=50)
    df['SMA_200'] = ta.sma(df['Close'], length=200)
    stoch = ta.stoch(df['High'], df['Low'], df['Close'], k=14, d=3, smooth_k=3)
    df = pd.concat([df, stoch], axis=1)
    
    k_col = [c for c in df.columns if 'STOCHk' in c][0]
    d_col = [c for c in df.columns if 'STOCHd' in c][0]

    current_row = df.iloc[-1]
    prev_row = df.iloc[-2]
    
    current_price = current_row['Close']
    k_today, d_today = current_row[k_col], current_row[d_col]
    k_yesterday, d_yesterday = prev_row[k_col], prev_row[d_col]
    
    # Technical Rules
    is_bullish_trend = current_price > current_row['SMA_50'] > current_row['SMA_200']
    was_oversold = k_yesterday < 20 or d_yesterday < 20
    is_crossover = k_yesterday <= d_yesterday and k_today > d_today

    # ----------------------------------------------------
    # STEP 2: Fetch Volatility Metrics via Tastytrade API
    # ----------------------------------------------------
    # get_market_metrics handles IV Rank and IV Percentile implicitly
    metrics = get_market_metrics(session, [ticker_symbol])[0]
    
    # Tastytrade handles percentages as decimals (e.g. 0.45 = 45%)
    iv_rank = float(metrics.implied_volatility_index_rank) * 100
    iv_percentile = float(metrics.implied_volatility_index_52_week_percentage) * 100
    current_iv = float(metrics.implied_volatility_index) * 100

    print(f"📊 {ticker_symbol} Volatility Profile:")
    print(f"   • Current IV: {current_iv:.1f}%")
    print(f"   • IV Rank: {iv_rank:.1f}")
    print(f"   • IV Percentile: {iv_percentile:.1f}")

    # ----------------------------------------------------
    # STEP 3: Validate Setup and Fetch Target Option Legs
    # ----------------------------------------------------
    if is_bullish_trend and was_oversold and is_crossover:
        print(f"✅ TRIGGER MET: Stochastic Crossover in Uptrend (%K: {k_today:.1f} > %D: {d_today:.1f})")
        
        # Volatility Filter: Optimal premium selling happens when IV Rank is healthy (>20-30+)
        if iv_rank < 15:
            print("⚠️ Warning: IV Rank is exceptionally low. Premium payout will be depressed.")

        find_optimal_legs(ticker_symbol, current_price)
    else:
        print(f"❌ Technical entry triggers not met. Skipping chain analysis.")


def find_optimal_legs(ticker_symbol, current_price):
    print("⛓️ Fetching live option chains from Tastytrade...")
    
    # Fetch active option chain structure
    chain = OptionChain.get_chain(session, ticker_symbol)
    
    # Target 30-45 Days to Expiration (DTE)
    today = datetime.today()
    min_date = today + timedelta(days=30)
    max_date = today + timedelta(days=45)
    
    # Find closest expiration cycle matching our DTE window
    target_exp = None
    for exp in chain.expirations:
        exp_date = datetime.strptime(exp.expiration_date, "%Y-%m-%d")
        if min_date <= exp_date <= max_date:
            target_exp = exp
            break
            
    if not target_exp:
        # Fallback to closest available expiration if exact window isn't filled
        target_exp = chain.expirations[0]
        
    print(f"   🎯 Selected Expiration Date: {target_exp.expiration_date} ({target_exp.days_to_expiration} DTE)")

    # Fetch all specific option contracts for this cycle containing Greeks
    nested_chain = chain.get_nested_chain(session, target_exp.expiration_date)
    
    # Filter out Put options
    put_legs = []
    for strike in nested_chain.strikes:
        if strike.put:
            # Native real-time Tastytrade Delta and Implied Vol per strike
            delta = abs(float(strike.put.delta)) if strike.put.delta else 0.0
            strike_price = float(strike.strike_price)
            
            put_legs.append({
                'strike': strike_price,
                'delta': delta,
                'symbol': strike.put.streamer_symbol
            })
            
    df_puts = pd.DataFrame(put_legs)
    
    # 📍 Find the ideal Short Put leg: Target Delta closest to 0.15 - 0.20
    target_delta = 0.15
    df_puts['delta_diff'] = (df_puts['delta'] - target_delta).abs()
    
    # Filter for OTM strikes (below current market price)
    otm_puts = df_puts[df_puts['strike'] < current_price]
    
    if otm_puts.empty:
        print("   ❌ No appropriate Out-of-the-Money Puts found.")
        return
        
    short_put_row = otm_puts.loc[otm_puts['delta_diff'].idxmin()]
    short_strike = short_put_row['strike']
    short_delta = short_put_row['delta']
    
    # 📍 Find the ideal Long Put leg: Standard 5-dollar wide protective wing
    long_strike_target = short_strike - 5
    df_puts['strike_diff'] = (df_puts['strike'] - long_strike_target).abs()
    long_put_row = df_puts.loc[df_puts['strike_diff'].idxmin()]
    long_strike = long_put_row['strike']
    long_delta = long_put_row['delta']

    print(f"\n🚀 PROPOSED BULL PUT SPREAD LEGS:")
    print(f"   1️⃣ SELL (Short) Strike: ${short_strike} Put  (Live Delta: {short_delta:.2f})")
    print(f"   2️⃣ BUY  (Long)  Strike: ${long_strike} Put   (Live Delta: {long_delta:.2f})")
    print(f"   💳 Capital Blocked (Width): ${int(short_strike - long_strike) * 100}")

# Example Execution
run_bull_put_scanner("AAPL")
```

---

# 💻 Expanded Python Core Script

Including two new strategies: Bear Call Spread (Bearish) and Iron Condor (Neutral)

📈 Market Regime Routing Logic
- Bearish Regime (Bear Call Spread): The price is trending below both SMAs, and the 50 SMA is below the 200 SMA (Death Cross). We wait for a temporary counter-trend rally where the Stochastic enters the overbought zone (>80) and turns back down. This allows us to sell call credit premium right at a short-term peak.
- Neutral Regime (Iron Condor): The price is flat-lining and weaving directly between the 50 SMA and 200 SMA, or the SMAs are completely flat. The Stochastic Oscillator moving cleanly between 20 and 80 signals a mean-reverting environment. We sell out-of-the-money options on both sides to capture pure premium decay.


```python
import yfinance as yf
import pandas as pd
import pandas_ta as ta
from datetime import datetime, timedelta
from tastytrade import ProductionSession
from tastytrade.metrics import get_market_metrics
from tastytrade.instruments import OptionChain

# 1. Authenticate with Tastytrade
USERNAME = "your_tastytrade_username"
PASSWORD = "your_tastytrade_password"

try:
    session = ProductionSession(USERNAME, PASSWORD)
    print("🔒 Tastytrade Session Authenticated Successfully.")
except Exception as e:
    print(f"❌ Login Failed: {e}")
    exit()

def analyze_market_regime(ticker_symbol):
    print(f"\n{"="*50}\n🔎 Scanning {ticker_symbol}...")
    
    # Fetch Daily Historical Stock Data
    stock = yf.Ticker(ticker_symbol)
    df = stock.history(period="1y", interval="1d")
    if len(df) < 200:
        print(f"❌ Not enough technical data for {ticker_symbol}.")
        return

    # Calculate Core Technical Indicators
    df['SMA_50'] = ta.sma(df['Close'], length=50)
    df['SMA_200'] = ta.sma(df['Close'], length=200)
    stoch = ta.stoch(df['High'], df['Low'], df['Close'], k=14, d=3, smooth_k=3)
    df = pd.concat([df, stoch], axis=1)
    
    k_col = [c for c in df.columns if 'STOCHk' in c][0]
    d_col = [c for c in df.columns if 'STOCHd' in c][0]

    current = df.iloc[-1]
    prev = df.iloc[-2]
    
    price = current['Close']
    sma50, sma200 = current['SMA_50'], current['SMA_200']
    k_today, d_today = current[k_col], current[d_col]
    k_yesterday, d_yesterday = prev[k_col], prev[d_col]

    # Fetch Volatility Metrics via Tastytrade API
    metrics = get_market_metrics(session, [ticker_symbol])[0]
    iv_rank = float(metrics.implied_volatility_index_rank) * 100
    print(f"📊 Volatility Profile -> Current Price: ${price:.2f} | IV Rank: {iv_rank:.1f}")

    # Determine Regime Structures
    is_bullish = price > sma50 > sma200
    is_bearish = price < sma50 < sma200
    is_neutral = (sma50 > price > sma200) or (sma200 > price > sma50) or (abs(sma50 - sma200) / sma200 < 0.02)

    # ----------------------------------------------------
    # ROUTING ENGINE
    # ----------------------------------------------------
    # 1️⃣ STRATEGY A: Bull Put Spread (Uptrend + Oversold Bounce)
    if is_bullish:
        stoch_oversold_crossover = (k_yesterday < 20 or d_yesterday < 20) and (k_yesterday <= d_yesterday and k_today > d_today)
        if stoch_oversold_crossover:
            print(f"✅ BULLISH TRIGGER: Preparing Bull Put Spread.")
            build_options_legs(ticker_symbol, price, strategy="BULL_PUT")
        else:
            print("⏳ Regime: Bullish, but waiting for an oversold Stochastic pullback.")

    # 2️⃣ STRATEGY B: Bear Call Spread (Downtrend + Overbought Peak)
    elif is_bearish:
        stoch_overbought_crossover = (k_yesterday > 80 or d_yesterday > 80) and (k_yesterday >= d_yesterday and k_today < d_today)
        if stoch_overbought_crossover:
            print(f"✅ BEARISH TRIGGER: Preparing Bear Call Spread.")
            build_options_legs(ticker_symbol, price, strategy="BEAR_CALL")
        else:
            print("⏳ Regime: Bearish, but waiting for an overbought Stochastic bounce.")

    # 3️⃣ STRATEGY C: Iron Condor (Range-bound sideways market)
    elif is_neutral:
        # For Iron Condors, we want to enter when IV Rank is elevated to capture volatility contraction
        if iv_rank > 25: 
            print(f"✅ NEUTRAL RANGE TRIGGER: Preparing Iron Condor Setup.")
            build_options_legs(ticker_symbol, price, strategy="IRON_CONDOR")
        else:
            print("⏳ Regime: Neutral, but IV Rank is too low to safely deploy an Iron Condor.")


def build_options_legs(ticker_symbol, current_price, strategy):
    chain = OptionChain.get_chain(session, ticker_symbol)
    
    # Target standard 30-45 DTE cycle
    today = datetime.today()
    target_exp = min(chain.expirations, key=lambda e: abs((datetime.strptime(e.expiration_date, "%Y-%m-%d") - today).days - 38))
    print(f"📅 Selected Cycle: {target_exp.expiration_date} ({target_exp.days_to_expiration} DTE)")

    nested_chain = chain.get_nested_chain(session, target_exp.expiration_date)
    
    # Flatten Strikes and extract Live Deltas
    options_list = []
    for strike in nested_chain.strikes:
        strike_price = float(strike.strike_price)
        call_delta = float(strike.call.delta) if strike.call and strike.call.delta else 0.0
        put_delta = abs(float(strike.put.delta)) if strike.put and strike.put.delta else 0.0
        
        options_list.append({
            'strike': strike_price,
            'call_delta': call_delta,
            'put_delta': put_delta
        })
    df_chain = pd.DataFrame(options_list)

    print(f"\n🚀 PROPOSED PRO-LEVEL LEGS FOR TYPE: {strategy}")

    # --- EXECUTE STRATEGY LOGIC ---
    if strategy == "BULL_PUT":
        short_put = df_chain[df_chain['strike'] < current_price].iloc[(df_chain['put_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_put = df_chain[df_chain['strike'] == (short_put['strike'] - 5.0)].iloc[0]
        print(f"   🟢 SELL $ {short_put['strike']} Put (Delta: {short_put['put_delta']:.2f})")
        print(f"   🔴 BUY  $ {long_put['strike']} Put  (Delta: {long_put['put_delta']:.2f})")

    elif strategy == "BEAR_CALL":
        # Target 0.15 Delta Call to sell
        short_call = df_chain[df_chain['strike'] > current_price].iloc[(df_chain['call_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_call = df_chain[df_chain['strike'] == (short_call['strike'] + 5.0)].iloc[0]
        print(f"   🟢 SELL $ {short_call['strike']} Call (Delta: {short_call['call_delta']:.2f})")
        print(f"   🔴 BUY  $ {long_call['strike']} Call  (Delta: {long_call['call_delta']:.2f})")

    elif strategy == "IRON_CONDOR":
        # Iron Condor combines an OTM Short Put and OTM Short Call, flanked by long protection wings
        short_put = df_chain[df_chain['strike'] < current_price].iloc[(df_chain['put_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_put = df_chain[df_chain['strike'] == (short_put['strike'] - 5.0)].iloc[0]
        
        short_call = df_chain[df_chain['strike'] > current_price].iloc[(df_chain['call_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_call = df_chain[df_chain['strike'] == (short_call['strike'] + 5.0)].iloc[0]
        
        print(f"   🟢 SELL $ {short_call['strike']} Call (Delta: {short_call['call_delta']:.2f})")
        print(f"   🔴 BUY  $ {long_call['strike']} Call  (Delta: {long_call['call_delta']:.2f})")
        print(f"   ➡️  --- Balanced Spreads ---")
        print(f"   🟢 SELL $ {short_put['strike']} Put  (Delta: {short_put['put_delta']:.2f})")
        print(f"   🔴 BUY  $ {long_put['strike']} Put   (Delta: {long_put['put_delta']:.2f})")

# Watchlist Loop Example
watchlist = ["AAPL", "AMD", "TLT", "SPY"]
for asset in watchlist:
    try:
        analyze_market_regime(asset)
    except Exception as err:
        print(f"⚠️ Error scanning {asset}: {err}")

```
---

# ⚙️ How to Automate Daily Execution

Running this script once a day, roughly 30 to 60 minutes before the market closes (around 3:00 PM to 3:30 PM EST), is the absolute sweet spot for this specific strategy.

```shell
# Example Cron Job: Runs the script Monday through Friday at 15:15 (3:15 PM) EST
15 15 * * 1-5 /usr/bin/python3 /path/to/your/options_script.py
```


> ### ⚠️ The One Exception
>
> Weekly Income ScannersIf your personal trading style shifts toward shorter-term trades (e.g., targeting weekly expirations that are only 7 to 14 Days to Expiration instead of 30-45 DTE), you can safely drop the timeframe down.If you choose to do this, change your charts to the 4-Hour Timeframe and change your trend filters from the 50/200 SMAs to the 20 SMA and 50 SMA. You can run the script twice a day: once at mid-day and once right before the close.


# Skip Earnings

Add the following method to check for Earnings

```python
def is_earnings_within_30_days(ticker_symbol):
    """
    Checks if the company has an upcoming earnings date scheduled within the next 30 days.
    """
    try:
        stock = yf.Ticker(ticker_symbol)
        calendar = stock.calendar
        
        # If yfinance calendar data is available, check for the next earnings date
        if calendar is not None and 'Earnings Date' in calendar:
            earnings_dates = calendar['Earnings Date']
            if earnings_dates:
                # Target the next upcoming date from the list
                next_earnings = earnings_dates[0]
                
                # Normalize datetimes to handle timezone-aware or naive structures cleanly
                now = datetime.now(timezone.utc) if next_earnings.tzinfo else datetime.now()
                days_until_earnings = (next_earnings - now).days
                
                # Check if it falls inside our high-risk 30-day window
                if 0 <= days_until_earnings <= 30:
                    print(f"⚠️ RISK DETECTED: {ticker_symbol} has earnings in {days_until_earnings} days ({next_earnings.strftime('%Y-%m-%d')}).")
                    return True
                elif days_until_earnings < 0:
                    # Date passed or occurred earlier today
                    return False
                else:
                    print(f"📅 Safe: Next earnings for {ticker_symbol} are in {days_until_earnings} days.")
                    return False
    except Exception as e:
        # Fallback to parsing the general earnings_dates dataframe if calendar fails
        try:
            stock = yf.Ticker(ticker_symbol)
            df_earnings = stock.get_earnings_dates(limit=1)
            if df_earnings is not None and not df_earnings.empty:
                next_earnings = df_earnings.index[0]
                now = datetime.now(timezone.utc) if next_earnings.tzinfo else datetime.now()
                days_until_earnings = (next_earnings - now).days
                if 0 <= days_until_earnings <= 30:
                    print(f"⚠️ RISK DETECTED: {ticker_symbol} has earnings in {days_until_earnings} days ({next_earnings.strftime('%Y-%m-%d')}).")
                    return True
        except Exception:
            print(f"ℹ️ Could not resolve earnings date for {ticker_symbol}. Proceeding with caution.")
    
    return False
```

Update method `analyze_market_regime` to check for Earnings:

```python
def analyze_market_regime(ticker_symbol):
    print(f"\n{"="*50}\n🔎 Scanning {ticker_symbol}...")
    
    # 🛑 NEW CRITICAL PROTECTION: The 30-day Catalyst Filter
    if is_earnings_within_30_days(ticker_symbol):
        print(f"⏭️ SKIPPING {ticker_symbol}: Avoid selling options premium right before a binary volatility event.")
        return
```

# 🔔 Telegram Notification

To send these alerts straight to your phone via Telegram, you will use a custom Telegram Bot. This allows the Python script to run silently on your machine or cloud server and instant-message you the exact option leg details only when a valid strategy trigger occurs.

### 🛠️ Step 1: Set Up Your Telegram Bot

Before changing your script, you need to create a bot and get your private keys:

1. Open Telegram and search for the user @BotFather.
2. Send the message /newbot and follow the prompts to give your bot a name and username.
3. Save the HTTP API Token provided by BotFather (it looks like 123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ).
4. Search for your newly created bot in Telegram and click Start / Send a message (this initializes the chat link).
5. To get your personal chat ID, search for @userinfobot in Telegram and send it a message. It will reply with your numeric Id (e.g., 987654321).

### 💻 Step 2: Add new method to send Telegram alert:

```python
# ----------------------------------------------------
# TELEGRAM CONFIGURATION
# ----------------------------------------------------
TELEGRAM_TOKEN = "YOUR_BOT_TOKEN_HERE"      # Paste token from BotFather
TELEGRAM_CHAT_ID = "YOUR_CHAT_ID_HERE"      # Paste ID from userinfobot

def send_telegram_alert(message):
    """Sends a formatted markdown text message directly to your Telegram chat."""
    url = f"https://telegram.org{TELEGRAM_TOKEN}/sendMessage"
    payload = {
        "chat_id": TELEGRAM_CHAT_ID,
        "text": message,
        "parse_mode": "Markdown"
    }
    try:
        response = requests.post(url, json=payload)
        if response.status_code == 200:
            print("📲 Telegram alert dispatched successfully!")
        else:
            print(f"⚠️ Telegram send failed: {response.text}")
    except Exception as e:
        print(f"❌ Error sending Telegram message: {e}")
```

### ⌨️ Step 3: Change method `build_option_legs` to create the message body and send to Telegram

```python
    # Format the base Telegram alert header
    alert_msg = f"🎯 *TRADE SIGNAL: {strategy}*\n"
    alert_msg += f"📈 *Asset:* {ticker_symbol} | *Price:* ${current_price:.2f}\n"
    alert_msg += f"📊 *IV Rank:* {iv_rank:.1f} | *Exp:* {target_exp.expiration_date} ({target_exp.days_to_expiration} DTE)\n"
    alert_msg += f"━━━━━━━━━━━━━━━━━━━━\n"

    if strategy == "BULL_PUT":
        short_put = df_chain[df_chain['strike'] < current_price].iloc[(df_chain['put_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_put = df_chain[df_chain['strike'] == (short_put['strike'] - 5.0)].iloc[0]
        
        alert_msg += f"🟢 *SELL:* ${short_put['strike']} Put (Δ {short_put['put_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_put['strike']} Put (Δ {long_put['put_delta']:.2f})\n"
        alert_msg += f"💳 *Margin Required:* ${int(short_put['strike'] - long_put['strike']) * 100}"

    elif strategy == "BEAR_CALL":
        short_call = df_chain[df_chain['strike'] > current_price].iloc[(df_chain['call_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_call = df_chain[df_chain['strike'] == (short_call['strike'] + 5.0)].iloc[0]
        
        alert_msg += f"🟢 *SELL:* ${short_call['strike']} Call (Δ {short_call['call_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_call['strike']} Call (Δ {long_call['call_delta']:.2f})\n"
        alert_msg += f"💳 *Margin Required:* ${int(long_call['strike'] - short_call['strike']) * 100}"

    elif strategy == "IRON_CONDOR":
        short_put = df_chain[df_chain['strike'] < current_price].iloc[(df_chain['put_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_put = df_chain[df_chain['strike'] == (short_put['strike'] - 5.0)].iloc[0]
        short_call = df_chain[df_chain['strike'] > current_price].iloc[(df_chain['call_delta'] - 0.15).abs().argsort()[:1]].iloc[0]
        long_call = df_chain[df_chain['strike'] == (short_call['strike'] + 5.0)].iloc[0]
        
        alert_msg += f"🟢 *SELL:* ${short_call['strike']} Call (Δ {short_call['call_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_call['strike']} Call (Δ {long_call['call_delta']:.2f})\n"
        alert_msg += f" ─── Wings ───\n"
        alert_msg += f"🟢 *SELL:* ${short_put['strike']} Put (Δ {short_put['put_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_put['strike']} Put (Δ {long_put['put_delta']:.2f})\n"
        alert_msg += f"💳 *Max Risk:* ${int(short_call['strike'] - long_call['strike']) * 100}"

    # Push to phone
    send_telegram_alert(alert_msg)
```

## 📱 What the Output Looks Like on Your Phone

When a stock triggers the execution parameters, the terminal remains quiet, and your phone will instantly buzz with a clean, Markdown-formatted push alert:

```text
🎯 TRADE SIGNAL: BULL_PUT
📈 Asset: AAPL | Price: $227.40
📊 IV Rank: 34.2 | Exp: 2026-10-23 (38 DTE)
━━━━━━━━━━━━━━━━━━━━
🟢 SELL: $215.00 Put (Δ 0.15)
🔴 BUY: $210.00 Put (Δ 0.08)
💳 Margin Required: $500
```

# 💻 Production Engine with Live Pricing & Risk Filtering

```python
import yfinance as yf
import pandas as pd
import pandas_ta as ta
import requests
from datetime import datetime, timedelta, timezone
from tastytrade import ProductionSession
from tastytrade.metrics import get_market_metrics
from tastytrade.instruments import OptionChain

# ----------------------------------------------------
# CONFIGURATION
# ----------------------------------------------------
TELEGRAM_TOKEN = "YOUR_BOT_TOKEN_HERE"
TELEGRAM_CHAT_ID = "YOUR_CHAT_ID_HERE"

# Premium Target Constraint: Minimum percentage of spread width to collect (0.30 = 30%)
MIN_PREMIUM_THRESHOLD_PCT = 0.30 

USERNAME = "your_tastytrade_username"
PASSWORD = "your_tastytrade_password"

try:
    session = ProductionSession(USERNAME, PASSWORD)
    print("🔒 Tastytrade Session Authenticated Successfully.")
except Exception as e:
    print(f"❌ Login Failed: {e}")
    exit()


def send_telegram_alert(message):
    url = f"https://telegram.org{TELEGRAM_TOKEN}/sendMessage"
    payload = {"chat_id": TELEGRAM_CHAT_ID, "text": message, "parse_mode": "Markdown"}
    try:
        requests.post(url, json=payload)
    except Exception as e:
        print(f"❌ Error sending Telegram message: {e}")


def is_earnings_within_30_days(ticker_symbol):
    try:
        stock = yf.Ticker(ticker_symbol)
        calendar = stock.calendar
        if calendar is not None and 'Earnings Date' in calendar:
            earnings_dates = calendar['Earnings Date']
            if earnings_dates:
                next_earnings = earnings_dates
                now = datetime.now(timezone.utc) if next_earnings.tzinfo else datetime.now()
                days_until_earnings = (next_earnings - now).days
                if 0 <= days_until_earnings <= 30:
                    return True, f"{days_until_earnings} days"
    except Exception:
        pass
    return False, ""


def analyze_market_regime(ticker_symbol):
    has_earnings, earnings_window = is_earnings_within_30_days(ticker_symbol)
    if has_earnings:
        print(f"⏭️ SKIPPING {ticker_symbol}: Earnings coming up in {earnings_window}")
        return

    stock = yf.Ticker(ticker_symbol)
    df = stock.history(period="1y", interval="1d")
    if len(df) < 200:
        return

    df['SMA_50'] = ta.sma(df['Close'], length=50)
    df['SMA_200'] = ta.sma(df['Close'], length=200)
    stoch = ta.stoch(df['High'], df['Low'], df['Close'], k=14, d=3, smooth_k=3)
    df = pd.concat([df, stoch], axis=1)
    
    k_col = [c for c in df.columns if 'STOCHk' in c]
    d_col = [c for c in df.columns if 'STOCHd' in c]

    current = df.iloc[-1]
    prev = df.iloc[-2]
    
    price = current['Close']
    sma50, sma200 = current['SMA_50'], current['SMA_200']
    k_today, d_today = current[k_col], current[d_col]
    k_yesterday, d_yesterday = prev[k_col], prev[d_col]

    metrics = get_market_metrics(session, [ticker_symbol])
    iv_rank = float(metrics.implied_volatility_index_rank) * 100

    is_bullish = price > sma50 > sma200
    is_bearish = price < sma50 < sma200
    is_neutral = (sma50 > price > sma200) or (sma200 > price > sma50) or (abs(sma50 - sma200) / sma200 < 0.02)

    # Route triggers directly to calculations
    if is_bullish and ((k_yesterday < 20 or d_yesterday < 20) and (k_yesterday <= d_yesterday and k_today > d_today)):
        build_and_filter_legs(ticker_symbol, price, iv_rank, strategy="BULL_PUT")
            
    elif is_bearish and ((k_yesterday > 80 or d_yesterday > 80) and (k_yesterday >= d_yesterday and k_today < d_today)):
        build_and_filter_legs(ticker_symbol, price, iv_rank, strategy="BEAR_CALL")
            
    elif is_neutral and iv_rank > 25:
        build_and_filter_legs(ticker_symbol, price, iv_rank, strategy="IRON_CONDOR")


def build_and_filter_legs(ticker_symbol, current_price, iv_rank, strategy):
    chain = OptionChain.get_chain(session, ticker_symbol)
    today = datetime.today()
    target_exp = min(chain.expirations, key=lambda e: abs((datetime.strptime(e.expiration_date, "%Y-%m-%d") - today).days - 38))

    nested_chain = chain.get_nested_chain(session, target_exp.expiration_date)
    options_list = []
    
    # 1. Gather strikes alongside their live bids and asks to derive real-time strategy mid-pricing
    for strike in nested_chain.strikes:
        strike_price = float(strike.strike_price)
        
        # Pull live greeks & quotes natively provided by Tastytrade objects
        call_delta = float(strike.call.delta) if strike.call and strike.call.delta else 0.0
        put_delta = abs(float(strike.put.delta)) if strike.put and strike.put.delta else 0.0
        
        call_mid = (float(strike.call.bid) + float(strike.call.ask)) / 2.0 if strike.call and strike.call.bid else 0.0
        put_mid = (float(strike.put.bid) + float(strike.put.ask)) / 2.0 if strike.put and strike.put.bid else 0.0
        
        options_list.append({
            'strike': strike_price,
            'call_delta': call_delta,
            'put_delta': put_delta,
            'call_mid': call_mid,
            'put_mid': put_mid
        })
        
    df_chain = pd.DataFrame(options_list)
    net_credit = 0.0
    spread_width = 5.0 # Set standard default spread width boundary

    # 2. Strategy Logic & Leg Selections
    if strategy == "BULL_PUT":
        short_put = df_chain[df_chain['strike'] < current_price].iloc[(df_chain['put_delta'] - 0.15).abs().argsort()[:1]].iloc
        long_put = df_chain[df_chain['strike'] == (short_put['strike'] - spread_width)].iloc
        
        # Net Credit for credit spreads = Short Option Premium Received - Long Option Premium Paid
        net_credit = short_put['put_mid'] - long_put['put_mid']
        
    elif strategy == "BEAR_CALL":
        short_call = df_chain[df_chain['strike'] > current_price].iloc[(df_chain['call_delta'] - 0.15).abs().argsort()[:1]].iloc
        long_call = df_chain[df_chain['strike'] == (short_call['strike'] + spread_width)].iloc
        
        net_credit = short_call['call_mid'] - long_call['call_mid']

    elif strategy == "IRON_CONDOR":
        short_put = df_chain[df_chain['strike'] < current_price].iloc[(df_chain['put_delta'] - 0.15).abs().argsort()[:1]].iloc
        long_put = df_chain[df_chain['strike'] == (short_put['strike'] - spread_width)].iloc
        short_call = df_chain[df_chain['strike'] > current_price].iloc[(df_chain['call_delta'] - 0.15).abs().argsort()[:1]].iloc
        long_call = df_chain[df_chain['strike'] == (short_call['strike'] + spread_width)].iloc
        
        # Iron Condor total credit combines both credit wings
        net_credit = (short_put['put_mid'] - long_put['put_mid']) + (short_call['call_mid'] - long_call['call_mid'])

    # 🛑 THE RISK PROFILE FILTER: Check mathematical viability
    required_min_credit = spread_width * MIN_PREMIUM_THRESHOLD_PCT
    
    if net_credit < required_min_credit:
        print(f"❌ FILTERED OUT: {ticker_symbol} {strategy} premium (${net_credit:.2f}) does not meet minimum 30% width rule (${required_min_credit:.2f}).")
        return

    # 3. Assemble and Format message for verified high-probability setups
    max_loss = (spread_width - net_credit) * 100
    alert_msg = f"🎯 *TRADE SIGNAL: {strategy}*\n"
    alert_msg += f"📈 *Asset:* {ticker_symbol} | *Price:* ${current_price:.2f}\n"
    alert_msg += f"📊 *IV Rank:* {iv_rank:.1f} | *Exp:* {target_exp.expiration_date} ({target_exp.days_to_expiration} DTE)\n"
    alert_msg += f"━━━━━━━━━━━━━━━━━━━━\n"

    if strategy == "BULL_PUT":
        alert_msg += f"🟢 *SELL:* ${short_put['strike']} Put (Δ {short_put['put_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_put['strike']} Put (Δ {long_put['put_delta']:.2f})\n"
    elif strategy == "BEAR_CALL":
        alert_msg += f"🟢 *SELL:* ${short_call['strike']} Call (Δ {short_call['call_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_call['strike']} Call (Δ {long_call['call_delta']:.2f})\n"
    elif strategy == "IRON_CONDOR":
        alert_msg += f"🟢 *SELL:* ${short_call['strike']} Call (Δ {short_call['call_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_call['strike']} Call (Δ {long_call['call_delta']:.2f})\n"
        alert_msg += f" ─── Wings ───\n"
        alert_msg += f"🟢 *SELL:* ${short_put['strike']} Put (Δ {short_put['put_delta']:.2f})\n"
        alert_msg += f"🔴 *BUY:* ${long_put['strike']} Put (Δ {long_put['put_delta']:.2f})\n"

    alert_msg += f"━━━━━━━━━━━━━━━━━━━━\n"
    alert_msg += f"💰 *Net Credit Received:* ${net_credit:.2f} ($ {net_credit*100:.0f} Total)\n"
    alert_msg += f"⚠️ *Max Defined Risk:* ${max_loss:.2f} per spread"

    # Push verification message straight to phone
    send_telegram_alert(alert_msg)

# Execution
watchlist = ["AAPL", "AMD", "MSFT", "NVDA", "SPY"]
for asset in watchlist:
    try:
        analyze_market_regime(asset)
    except Exception as err:
        print(f"Error on {asset}: {err}")

```