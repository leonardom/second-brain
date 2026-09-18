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