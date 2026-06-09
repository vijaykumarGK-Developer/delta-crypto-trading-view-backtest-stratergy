<p align="center">
  <img src="https://img.shields.io/badge/Python-3.8%2B-3776AB?style=for-the-badge&logo=python" alt="Python">
  <img src="https://img.shields.io/badge/Framework-backtrader-0097A7?style=for-the-badge" alt="Framework">
  <img src="https://img.shields.io/badge/Exchange-Delta%20v2-00BFA5?style=for-the-badge" alt="Exchange">
  <img src="https://img.shields.io/badge/Notebook-Jupyter-F37626?style=for-the-badge&logo=jupyter" alt="Jupyter">
  <img src="https://img.shields.io/github/license/vijaykumarGK-Developer/delta-crypto-trading-view-backtest-stratergy?style=for-the-badge" alt="License">
</p>

<h1 align="center">📊 Delta Crypto TradingView-Style Backtesting & Automation Framework</h1>
<h3 align="center">A free, Python-powered alternative to TradingView Pine Script for Delta Exchange</h3>

<p align="center">
  Backtest complex crypto trading strategies on years of 1-minute data — without paying for TradingView Premium.<br>
  Built with <code>backtrader</code>, <code>pandas</code>, <code>plotly</code>, and Delta Exchange's v2 API.
</p>

---

## 📋 Table of Contents

- [Why This Framework?](#-why-this-framework)
- [Strategies Included](#-strategies-included)
- [Modular Framework Structure](#-modular-framework-structure)
- [Project Structure](#-project-structure)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Configuration Reference](#-configuration-reference)
- [Backtest Results](#-backtest-results)
- [Automating Your Strategy](#-automating-your-strategy)
- [Contributing](#-contributing)
- [License](#-license)
- [Disclaimer](#-disclaimer)

---

## 🌟 Why This Framework?

| Feature | TradingView Pine Script | This Framework |
|---|---|---|
| **Cost** | $49–$155/month (Premium required for backtesting) | **Free** |
| **Data Access** | Limited by subscription tier | Unlimited — pull years of 1-min data via API |
| **Library Support** | Pine Script only (no external libs) | **Full Python ecosystem**: TA-Lib, Scikit-Learn, NumPy, Pandas |
| **Backtesting Engine** | Built-in (opaque) | **backtrader** — transparent, customizable, event-driven |
| **Live Automation** | Requires separate infrastructure | Same code plugs directly into Delta API |
| **Order Types** | Limited | Full support: TP, SL, trailing stops, time-based exits |
| **Charts** | TradingView charts (locked to platform) | **Plotly** — interactive, exportable, zoomable |

---

## 📈 Strategies Included

| Notebook | Strategy | Core Logic |
|---|---|---|
| `Day_Trading_Strategy.ipynb` | VWAP Mean Reversion Day Trade | Enters when price deviates from VWAP by N standard deviations; exits at TP/SL or EOD |
| `VWAP_Trading_Strategy.ipynb` | Multi-Indicator VWAP Strategy | VWAP + SMA crossover + RSI filter + Bollinger Band confluence |
| `Untitled-1.ipynb` | Custom / Experimental | Template for building your own strategy |

---

## 🏗 Modular Framework Structure

The framework is organized into **7 Logical Blocks** that execute sequentially within each Jupyter notebook:

```
┌─────────────────────────────────────────────────────────────────┐
│                     JUPYTER NOTEBOOK FLOW                        │
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                   │
│  │ BLOCK 1  │───>│ BLOCK 2  │───>│ BLOCK 3  │                   │
│  │  API     │    │ Strategy │    │  Tech    │                   │
│  │  Auth    │    │ Config   │    │  Ind.    │                   │
│  └──────────┘    └──────────┘    └──────────┘                   │
│       │              │               │                          │
│       ▼              ▼               ▼                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                   │
│  │ BLOCK 4  │───>│ BLOCK 5  │───>│ BLOCK 6  │                   │
│  │  Data    │    │backtrader│    │ Analysis │                   │
│  │  Engine  │    │ Execution│    │ & Logs   │                   │
│  └──────────┘    └──────────┘    └──────────┘                   │
│       │              │               │                          │
│       ▼              ▼               ▼                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  BLOCK 7: Interactive Visualizations (Plotly)           │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 🔐 Block 1: API Authentication

Implements **HMAC-SHA256** signature generation for secure communication with Delta Exchange. Pre-configured for both:
- `delta.exchange` (global)
- `india.delta.exchange` (Indian users)

**Key functions:**
- `generate_signature()` — Creates HMAC-SHA256 payload signature
- `delta_request()` — Authenticated HTTP request wrapper

### ⚙️ Block 2: Strategy Configuration

Central configuration hub for all trading parameters:

| Parameter | Options | Description |
|---|---|---|
| `MODE` | `LONG_ONLY`, `SHORT_ONLY`, `BOTH` | Directional bias |
| `SYMBOL` | `BTCUSD`, `ETHUSD`, etc. | Trading pair |
| `ENTRY_START` / `ENTRY_END` | HH:MM (IST/UTC) | Entry time window |
| `EXIT_TIME` | HH:MM (IST/UTC) | End-of-day exit |
| `TP_POINTS` / `TP_PERCENT` | Float | Take profit target |
| `SL_POINTS` / `SL_PERCENT` | Float | Stop loss level |
| `COMMISSION` | Float (e.g., 0.05) | Broker commission % |
| `SLIPPAGE` | Float (e.g., 0.01) | Slippage % per trade |

### 📊 Block 3: Technical Indicator Logic

| Indicator | Period / Config | Source |
|---|---|---|
| **VWAP** | Session / Week / Month anchors | Custom calculation |
| VWAP Confidence Bands | ±1σ, ±2σ, ±3σ | Standard deviation |
| SMA | 9, 21, 50 periods | TA-Lib / NumPy |
| RSI | 14 periods | TA-Lib / NumPy |
| Bollinger Bands | 20, 2σ | TA-Lib / NumPy |
| ATR | 14 periods | TA-Lib / NumPy |

### 📥 Block 4: Data Acquisition Engine

- **Source:** Delta Exchange v2 market data API
- **Resolution:** 1-minute OHLCV candles
- **Duration:** 180+ days
- **Caching:** Saves to `.pkl` files for subsequent runs (no redundant API calls)
- **Fallback:** Uses cached data if API is unavailable

### 🚀 Block 5: Backtrader Execution Engine

- **Framework:** `backtrader` — industry-standard event-driven backtesting
- **Order Types:** Market, Limit, Stop, StopLimit
- **Position Sizing:** Fixed quantity or percentage-based
- **Metrics calculated:**
  - Total Return (%)
  - Win Rate (%)
  - Profit Factor
  - Maximum Drawdown (%)
  - Sharpe Ratio
  - Total Trades
  - Average Trade Duration

### 📜 Block 6: TradingView-Style Analysis

- **Trade Log:** Every entry/exit with timestamp, price, size, P&L, and exit reason
- **Exit Reasons:** `TAKE_PROFIT`, `STOP_LOSS`, `END_OF_DAY`, `SIGNAL_REVERSE`
- **Performance Summary:** Single-page strategy health dashboard
- **Rainbow Formatting:** Color-coded output for readability

### 📈 Block 7: Interactive Visualizations

- **Library:** Plotly (high-performance, WebGL-accelerated)
- **Candlestick Chart:** Interactive, zoomable, scrollable
- **Indicator Overlays:** VWAP, SMA, Bollinger Bands, Confidence Bands
- **Trade Markers:** Green triangles (Buys), Red triangles (Sells)
- **Export:** Charts can be saved as HTML or PNG

---

## 📂 Project Structure

```
delta-crypto-trading-view-backtest-stratergy/
├── Day_Trading_Strategy.ipynb      # VWAP Mean Reversion Day Trading Strategy
├── VWAP_Trading_Strategy.ipynb     # Multi-Indicator VWAP Strategy
├── Untitled-1.ipynb                # Custom strategy template
├── README.md                       # You are here
├── .gitignore                      # Python + Jupyter ignores
└── *.pkl                           # Cached market data (generated, not committed)
```

---

## 🛠 Tech Stack

| Technology | Purpose |
|---|---|
| **Python 3.8+** | Core language |
| **backtrader** | Event-driven backtesting engine |
| **pandas** | Data manipulation and analysis |
| **numpy** | Numerical computing |
| **requests** | HTTP client for Delta Exchange API |
| **plotly** | Interactive visualization |
| **TA-Lib** (optional) | Technical analysis indicators |
| **Delta Exchange v2 API** | Market data and trading |
| **Jupyter Notebook** | Interactive development environment |

---

## 🚀 Getting Started

### Prerequisites

- Python 3.8 or higher
- A [Delta Exchange](https://www.delta.exchange/) account with API credentials
- Jupyter (install via `pip install jupyter`)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/vijaykumarGK-Developer/delta-crypto-trading-view-backtest-stratergy.git
cd delta-crypto-trading-view-backtest-stratergy

# 2. Install dependencies
pip install pandas numpy requests plotly backtrader jupyter

# 3. (Optional) Install TA-Lib for accelerated indicators
#    Follow: https://github.com/mrjbq7/ta-lib#installation

# 4. Launch Jupyter
jupyter notebook
```

### Configuration

1. Open `Day_Trading_Strategy.ipynb` in Jupyter.
2. Navigate to **Block 1** and add your Delta Exchange credentials:
   ```python
   API_KEY = "your_api_key_here"
   API_SECRET = "your_api_secret_here"
   ```
3. In **Block 2**, configure your strategy:
   ```python
   SYMBOL = "BTCUSD"
   MODE = "BOTH"
   TP_PERCENT = 0.5
   SL_PERCENT = 0.3
   ```

### Execution

Run each block sequentially (`Shift+Enter`):

| Step | Block | Action |
|---|---|---|
| 1 | Block 1 | Authenticate with Delta Exchange |
| 2 | Block 2 | Set strategy parameters |
| 3 | Block 3 | Define technical indicators |
| 4 | Block 4 | Fetch and cache historical data |
| 5 | Block 5 | Run backtrader simulation |
| 6 | Block 6 | Review trade logs and performance |
| 7 | Block 7 | Explore interactive charts |

> **Pro Tip:** After modifying parameters in Block 2, re-run from Block 5 (skip data fetching to use cached `.pkl` files).

---

## ⚙️ Configuration Reference

### Trading Modes

```python
MODE = "BOTH"        # Trade both directions
MODE = "LONG_ONLY"   # Long positions only
MODE = "SHORT_ONLY"  # Short positions only
```

### Risk Management Parameters

```python
# Take Profit
TP_POINTS = 50       # Exit after 50 points profit (futures)
TP_PERCENT = 0.5     # Exit after 0.5% profit (spot)

# Stop Loss
SL_POINTS = 30       # Exit after 30 points loss
SL_PERCENT = 0.3     # Exit after 0.3% loss

# Costs
COMMISSION = 0.05    # 0.05% commission per trade
SLIPPAGE = 0.01      # 0.01% slippage
```

### Time Controls

```python
ENTRY_START = "09:15"   # Start entry window (IST)
ENTRY_END = "14:30"     # Close entry window (IST)
EXIT_TIME = "15:15"     # Force exit all positions (EOD)
```

---

## 📊 Backtest Results

When you run a strategy, Block 6 outputs a comprehensive performance summary:

```
╔══════════════════════════════════════════╗
║        PERFORMANCE SUMMARY              ║
╠══════════════════════════════════════════╣
║  Total Return        : +12.45%          ║
║  Win Rate            : 68.42%           ║
║  Profit Factor       : 2.31             ║
║  Total Trades        : 38               ║
║  Max Drawdown        : -4.21%           ║
║  Sharpe Ratio        : 1.87             ║
║  Avg Trade Duration  : 2h 34m           ║
╚══════════════════════════════════════════╝
```

And a detailed trade log:

```
┌──────┬──────────┬──────────┬───────┬────────┬──────────┬──────────────┐
│ Trade│  Entry   │  Exit    │  P&L  │ Return │ Exit     │  Duration    │
│  #   │  Time    │  Time    │  ($)  │  (%)   │ Reason   │              │
├──────┼──────────┼──────────┼───────┼────────┼──────────┼──────────────┤
│  1   │ 09:45    │ 11:20    │ +45   │ +0.8%  │ TP       │ 1h 35m       │
│  2   │ 10:30    │ 10:45    │ -22   │ -0.3%  │ SL       │ 0h 15m       │
│  3   │ 13:15    │ 15:15    │ +18   │ +0.3%  │ EOD      │ 2h 00m       │
└──────┴──────────┴──────────┴───────┴────────┴──────────┴──────────────┘
```

---

## 🤖 Automating Your Strategy

This framework is designed as the **first step toward a live trading bot**. The same logic extends seamlessly:

```python
# 1. Fetch latest candle (from Block 4)
latest = fetch_latest_candle(symbol)

# 2. Generate signal (from Block 3)
signal = check_entry_conditions(latest, indicators)

# 3. Place order (from Block 1 signature logic)
if signal == "BUY":
    order = place_order(symbol, "buy", quantity, order_type="market")
elif signal == "SELL":
    order = place_order(symbol, "sell", quantity, order_type="market")
```

### Recommended Automation Architecture

```
┌──────────────┐     ┌─────────────────┐     ┌────────────────┐
│  Backtest    │────>│  Paper Trade    │────>│  Live Trade    │
│  (This Repo) │     │  (Simulated)    │     │  (Real Funds)  │
└──────────────┘     └─────────────────┘     └────────────────┘
        │                     │                       │
        │                     │                       │
    Validate              Validate               Execute
    Strategy              Execution              Live Orders
    Logic                 + Timing
```

---

## 🤝 Contributing

Contributions are welcome! Here's how to get involved:

### Workflow

1. **Fork** the repository.
2. **Create a feature branch:**
   ```bash
   git checkout -b feature/new-strategy
   ```
3. **Add your strategy** as a new `.ipynb` notebook following the 7-block structure.
4. **Test** with at least 90 days of historical data.
5. **Commit:**
   ```bash
   git commit -m "feat: add [Strategy Name] with [indicator] logic"
   ```
6. **Push:**
   ```bash
   git push origin feature/new-strategy
   ```
7. **Open a Pull Request.**

### Guidelines

- Follow the existing 7-block notebook structure.
- Document all configuration parameters clearly.
- Include a performance summary in the PR description.
- Add TA-Lib or NumPy fallbacks for indicators.
- Ensure all API credentials are referenced via variables (never hardcoded).

### Ideas for Contributions

- [ ] Adding more exchanges (Binance, Bybit, Coinbase)
- [ ] Machine learning-based signal generation (Scikit-Learn)
- [ ] Portfolio-level multi-asset backtesting
- [ ] Telegram/Discord alert integration
- [ ] Docker containerization for cloud deployment
- [ ] Web dashboard (Streamlit/Gradio)

---

## 📄 License

Distributed under the **MIT License**. See [LICENSE](LICENSE) for more information.

---

## ⚠️ Disclaimer

> **IMPORTANT:** This software is provided for **educational and research purposes only**.
>
> - Trading cryptocurrency carries **significant financial risk**.
> - Past performance in backtests does **not** guarantee future results.
> - Always test strategies in a paper trading environment before using real funds.
> - Never trade more than you can afford to lose.
> - The authors are not financial advisors and accept no liability for any losses incurred.

---

<p align="center">
  <b>Delta Crypto TradingView-Style Backtesting & Automation Framework</b>
  <br>
  <i>Backtest like a pro — without paying for Premium.</i>
  <br><br>
  <a href="https://github.com/vijaykumarGK-Developer/delta-crypto-trading-view-backtest-stratergy">GitHub</a> •
  <a href="https://www.delta.exchange/">Delta Exchange</a> •
  <a href="https://github.com/mementum/backtrader">backtrader</a>
  <br><br>
  <sub>Built with Python, backtrader, and Plotly</sub>
</p>
