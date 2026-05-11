# Delta Crypto TradingView-Style Backtesting & Automation Framework

A professional-grade, Python-based trading framework designed to provide a free and flexible alternative to TradingView's Pine Script. This framework is specifically built for **Delta Exchange**, allowing users to backtest complex day trading and mean reversion strategies using industrial-strength libraries like `backtrader`.

## 🌟 Why This Framework?

- **Zero Cost:** No need for TradingView Premium; backtest over years of 1-minute data for free.
- **Python Power:** Use any library (TA-Lib, Scikit-Learn, Pandas) which is impossible in Pine Script.
- **Delta Exchange Native:** Direct integration with Delta Exchange's v2 API for data and signature-based authentication.
- **Seamless Automation:** Once a strategy is validated, the same logic can be plugged into the Delta API for live execution.

---

## 🏗 Modular Framework Structure

The framework is organized into **7 Logical Blocks** that must be executed sequentially:

### 🔐 BLOCK 1: API Authentication
- **Security:** Implements **HMAC-SHA256** signature generation for secure communication.
- **Customization:** Add your `API_KEY` and `API_SECRET`.
- **Endpoints:** Pre-configured for both `delta.exchange` and `india.delta.exchange`.

### ⚙️ BLOCK 2: Strategy Configuration
- **Trading Modes:** Toggle between `LONG_ONLY`, `SHORT_ONLY`, or `BOTH`.
- **Time Controls:** Precise entry/exit times (IST/UTC), ideal for "Midnight Entry" or specific session strategies.
- **Risk Management:** 
    - **Take Profit (TP):** Points or Percentage-based.
    - **Stop Loss (SL):** Points or Percentage-based.
    - **Costs:** Configurable commission and slippage for realistic results.

### 📊 BLOCK 3: Technical Indicator Logic
- **Primary Indicators:** VWAP with customizable confidence bands (Session/Week/Month anchors).
- **Secondary Indicators:** SMA (9, 21, 50), RSI (14), Bollinger Bands, and ATR.
- **Libraries:** Utilizes `TA-Lib` for speed, with `NumPy` fallbacks.

### 📥 BLOCK 4: Data Acquisition Engine
- **API Fetching:** Automatically pulls OHLCV data from Delta Exchange v2 history endpoints.
- **Smart Caching:** Saves data to local `.pkl` files to avoid redundant API calls and speed up subsequent runs.
- **Scalability:** Handles 180+ days of 1-minute data effortlessly.

### 🚀 BLOCK 5: Backtrader Execution Engine
- **Event-Driven:** Powered by the `backtrader` library.
- **Simulation:** Executes trades based on Block 3 logic while respecting Block 2 parameters.
- **Metrics:** Calculates Win Rate, Profit Factor, and Maximum Drawdown.

### 📜 BLOCK 6: TradingView-Style Analysis
- **Comprehensive Logs:** Generates detailed trade lists with entry/exit timestamps, prices, and **Exit Reasons** (TP, SL, or EOD).
- **Performance Summary:** A high-level overview of strategy health.
- **Rainbow Formatting:** Color-coded CSV-style logs for easy readability.

### 📈 BLOCK 7: Interactive Visualizations
- **Plotly Integration:** High-performance, scrollable candlestick charts.
- **Overlays:** Indicators (VWAP, SMA) and Confidence Bands plotted directly on price.
- **Signals:** Buy and Sell markers clearly visualized to audit strategy behavior.

---

## 🚀 Getting Started

### 1. Prerequisites
Ensure you have Python 3.8+ installed. Install the required dependencies:
```bash
pip install pandas numpy requests plotly backtrader
```
*Note: If you want to use TA-Lib, follow the [installation guide for your OS](https://github.com/mrjbq7/ta-lib).*

### 2. Configuration
1. Open the `.ipynb` file of your choice (e.g., `Day_Trading_Strategy.ipynb`).
2. In **Block 1**, paste your Delta Exchange credentials.
3. In **Block 2**, adjust your symbol (e.g., `BTCUSD`) and risk parameters.

### 3. Execution
Run each block in the Jupyter Notebook sequentially. If you modify parameters in Block 2, you only need to re-run from Block 5 onwards.

---

## 🤖 Automating Your Strategy
This framework is the first step toward a trading bot. By leveraging the authentication logic in Block 1 and the signal generation in Block 3, you can easily extend this script to:
1. Fetch the latest candle.
2. Check for a signal.
3. Place an order via `requests.post` to the Delta Exchange order endpoint.

---
*Disclaimer: This software is for educational purposes. Trading cryptocurrency carries significant risk. Never trade more than you can afford to lose.*
