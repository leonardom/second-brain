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