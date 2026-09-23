# Example of Stock Option Payoff Chart

```python
import matplotlib.pyplot as plt
import numpy as np

# 1. Define Option Parameters
strike_price = 100.00  # Exercise price (K)
premium = 5.00  # Cost of the option
break_even = strike_price + premium

# 2. Generate a range of spot prices at expiration
# We generate prices from $80 to $120 to clearly see the behavior around the strike
spot_prices = np.linspace(80, 120, 100)

# 3. Calculate Payoff (Profit / Loss)
# np.maximum(x, 0) simulates the max(S - K, 0) intrinsic value formula
intrinsic_value = np.maximum(spot_prices - strike_price, 0)
payoff = intrinsic_value - premium

# 4. Create the Plot
plt.figure(figsize=(10, 6))

# Plot the payoff line
plt.plot(spot_prices, payoff, label="Long Call Payoff", color="#1f77b4", lw=2.5)

# Add baseline reference lines
plt.axhline(0, color="black", linestyle="--", alpha=0.7, label="Break-even Baseline")
plt.axvline(strike_price, color="red", linestyle=":", alpha=0.7, label=f"Strike Price (${strike_price})")

# Highlight key points
plt.scatter([strike_price, break_even], [-premium, 0], color="orange", zorder=5)
plt.text(strike_price + 0.5, -premium - 1, f"Strike (${strike_price})", fontsize=9, color="red")
plt.text(break_even + 0.5, 1, f"Break-even (${break_even})", fontsize=9, color="green")

# Shading Profit and Loss regions
plt.fill_between(spot_prices, payoff, 0, where=(payoff > 0), facecolor="green", alpha=0.15)
plt.fill_between(spot_prices, payoff, 0, where=(payoff < 0), facecolor="red", alpha=0.15)

# Labels and Styling
plt.title("Long Call Option Payoff at Expiration", fontsize=14, fontweight="bold")
plt.xlabel("Stock Price at Expiration ($)", fontsize=11)
plt.ylabel("Profit / Loss ($)", fontsize=11)
plt.grid(True, linestyle=":", alpha=0.6)
plt.legend(loc="upper left")

# Show the plot
plt.show()
```

## Icon Condor Example

```python
import matplotlib.pyplot as plt
import numpy as np

# 1. Define Option Parameters (4 Legs)
# Long Put (A), Short Put (B), Short Call (C), Long Call (D)
k_long_put = 90.00
k_short_put = 95.00
k_short_call = 105.00
k_long_call = 110.00

# Premiums collected (+) or paid (-)
premium_long_put = -1.00
premium_short_put = 3.50
premium_short_call = 2.00
premium_long_call = -1.50

# Total Net Premium Collected (Maximum Profit)
net_premium = (
    premium_long_put + premium_short_put + premium_short_call + premium_long_call
)

# Break-even Points
be_lower = k_short_put - net_premium
be_upper = k_short_call + net_premium

# 2. Generate range of stock prices at expiration
spot_prices = np.linspace(70, 130, 500)


# 3. Payoff Functions for individual legs
def call_payoff(s, k, premium, position="long"):
    intrinsic = np.maximum(s - k, 0)
    return (intrinsic + premium) if position == "long" else (-intrinsic + premium)


def put_payoff(s, k, premium, position="long"):
    intrinsic = np.maximum(k - s, 0)
    return (intrinsic + premium) if position == "long" else (-intrinsic + premium)


# 4. Calculate Net Payoff by summing all 4 legs
payoff = (
    put_payoff(spot_prices, k_long_put, premium_long_put, "long")
    + put_payoff(spot_prices, k_short_put, premium_short_put, "short")
    + call_payoff(spot_prices, k_short_call, premium_short_call, "short")
    + call_payoff(spot_prices, k_long_call, premium_long_call, "long")
)

# 5. Create the Plot
plt.figure(figsize=(10, 6))

# Plot the total Iron Condor Payoff
plt.plot(spot_prices, payoff, label="Iron Condor Payoff", color="#9467bd", lw=2.5)

# Add baseline reference lines
plt.axhline(0, color="black", linestyle="--", alpha=0.5)

# Add vertical lines for all 4 strike prices
strikes = [k_long_put, k_short_put, k_short_call, k_long_call]
colors = ["#ff7f0e", "#2ca02c", "#2ca02c", "#ff7f0e"]
labels = ["Long Put", "Short Put", "Short Call", "Long Call"]

for strike, color, label in zip(strikes, colors, labels):
    plt.axvline(strike, color=color, linestyle=":", alpha=0.6)
    plt.text(
        strike - 1.5,
        np.min(payoff) - 0.8,
        f"{label}\n${strike}",
        fontsize=8,
        color=color,
    )

# Highlight Break-even Points
plt.scatter([be_lower, be_upper], [0, 0], color="red", zorder=5)
plt.text(be_lower - 5, 0.4, f"BE: ${be_lower:.2f}", fontsize=9, color="black")
plt.text(be_upper + 1, 0.4, f"BE: ${be_upper:.2f}", fontsize=9, color="black")

# Shading Profit and Loss regions
plt.fill_between(spot_prices, payoff, 0, where=(payoff > 0), facecolor="green", alpha=0.15)
plt.fill_between(spot_prices, payoff, 0, where=(payoff < 0), facecolor="red", alpha=0.15)

# Labels and Styling
plt.title("Iron Condor Strategy Payoff at Expiration", fontsize=14, fontweight="bold")
plt.xlabel("Stock Price at Expiration ($)", fontsize=11)
plt.ylabel("Profit / Loss ($)", fontsize=11)
plt.grid(True, linestyle=":", alpha=0.6)

plt.xlim(70, 130)
plt.ylim(np.min(payoff) - 1.5, np.max(payoff) + 1.5)

# Show the plot
plt.show()
```