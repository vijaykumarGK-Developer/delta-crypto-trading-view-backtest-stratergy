# 📖 Delta Crypto TradingView-Style Backtesting & Automation Framework — Documentation

> **Version:** 1.0.0 | **Last Updated:** June 2026 | **Language:** Python 3.8+ | **Engine:** backtrader

---

## Table of Contents

1. [Project Overview](#-project-overview)
2. [Architecture & System Design](#-architecture--system-design)
3. [Notebook Reference](#-notebook-reference)
4. [Block-by-Block Technical Reference](#-block-by-block-technical-reference)
5. [API Reference](#-api-reference)
6. [Strategy Configuration Reference](#-strategy-configuration-reference)
7. [Technical Indicators Library](#-technical-indicators-library)
8. [Backtrader Engine Deep Dive](#-backtrader-engine-deep-dive)
9. [Data Pipeline](#-data-pipeline)
10. [Performance Metrics & Trade Analysis](#-performance-metrics--trade-analysis)
11. [Plotly Visualization Guide](#-plotly-visualization-guide)
12. [Live Trading Automation](#-live-trading-automation)
13. [Scalping Optimization Analysis](#-scalping-optimization-analysis)
14. [Contributing Guidelines](#-contributing-guidelines)
15. [Code of Conduct](#-code-of-conduct)
16. [Security Policy](#-security-policy)
17. [Changelog](#-changelog)
18. [FAQ](#-faq)
19. [Troubleshooting Guide](#-troubleshooting-guide)
20. [Disclaimer](#-disclaimer)

---

## 📋 Project Overview

### What Is This Framework?

A professional-grade, Python-based cryptocurrency trading backtesting framework designed as a **free, open-source alternative to TradingView's Pine Script**. It integrates directly with the **Delta Exchange v2 API** and leverages the `backtrader` event-driven backtesting engine for realistic trade simulation.

### Why Build This?

| Problem | Solution |
|---|---|
| TradingView Premium costs **$49–$155/month** for backtesting | **Free** — no subscription needed |
| Pine Script cannot use external libraries | **Full Python ecosystem** — TA-Lib, Scikit-Learn, Pandas, NumPy |
| Limited to 20k bars in Pine Script | **Unlimited** — years of 1-minute data via Delta API |
| Opaque backtesting logic (black box) | **Transparent** — event-driven `backtrader` engine |
| Difficult to automate | **Seamless** — same code works for live trading |

### What Can You Do With It?

- Backtest **day trading** and **mean reversion** strategies on 180+ days of 1-minute OHLCV data
- Simulate realistic trading with configurable **commission, slippage, take profit, and stop loss**
- Generate **TradingView-style trade logs** with exit reason analysis
- Create **interactive Plotly candlestick charts** with indicator overlays and trade markers
- Analyze **scalping optimization** for daily profit targets
- Extend to **live trading automation** via the Delta Exchange API

---

## 🏗 Architecture & System Design

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      DELTA CRYPTO TRADING FRAMEWORK                      │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    JUPYTER NOTEBOOK LAYER                        │   │
│  │                                                                  │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐│   │
│  │  │  Block 1 │  │  Block 2 │  │  Block 3 │  │  Block 4         ││   │
│  │  │  API     │─>│  Strategy│─>│  Tech    │─>│  Data            ││   │
│  │  │  Auth    │  │  Config  │  │  Ind.    │  │  Acquisition     ││   │
│  │  └──────────┘  └──────────┘  └──────────┘  └────────┬─────────┘│   │
│  │                                                      │           │   │
│  │  ┌───────────────────────────────────────────────────▼────────┐  │   │
│  │  │                    Block 5: backtrader Engine                │  │   │
│  │  │  ┌─────────────────────────────────────────────────────┐   │  │   │
│  │  │  │  Cerebro Engine  ──>  Strategy Class  ──>  Analyers  │   │  │   │
│  │  │  │  (Event Loop)           (Trade Logic)     (Metrics)  │   │  │   │
│  │  │  └─────────────────────────────────────────────────────┘   │  │   │
│  │  └────────────────────────────────────────────────────────────┘  │   │
│  │                                                                  │   │
│  │  ┌──────────┐  ┌──────────┐                                      │   │
│  │  │  Block 6 │  │  Block 7 │                                      │   │
│  │  │  Trade   │  │  Plotly  │                                      │   │
│  │  │  Logs    │  │  Charts  │                                      │   │
│  │  └──────────┘  └──────────┘                                      │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    DATA & EXTERNAL LAYER                         │   │
│  │                                                                  │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌──────────────────┐   │   │
│  │  │  Delta Exchange │  │  Local Cache   │  │  TA-Lib / NumPy  │   │   │
│  │  │  v2 API        │  │  (*.pkl files) │  │  Indicators      │   │   │
│  │  └────────────────┘  └────────────────┘  └──────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### Data Flow

```
Delta Exchange API (v2/history/candles)
        │
        ▼
┌───────────────────┐
│  Block 4: Data    │
│  Acquisition      │
│                   │
│  HTTP GET request │
│  (batched by day) │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Local Cache      │
│  (*.pkl)          │
│                   │
│  pickle.dump/load │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  pandas DataFrame │
│                   │
│  Columns:         │
│  timestamp, open, │
│  high, low, close,│
│  volume           │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  backtrader Feed  │
│                   │
│  bt.feeds.Pandas- │
│  DataFrame        │
└─────────┬─────────┘
          │
          ▼
┌────────────────────────────────────┐
│  Block 5: Cerebro Event Loop       │
│                                    │
│  For each bar (1-min interval):    │
│    1. Update indicators (SMA, RSI, │
│       BB, VWAP, ATR)              │
│    2. Check exit conditions (TP,   │
│       SL, EOD, signal)            │
│    3. Check entry conditions       │
│    4. Execute orders               │
│    5. Record trade metrics         │
└─────────────────┬──────────────────┘
                  │
                  ▼
┌───────────────────┐  ┌───────────────────┐
│  Block 6:         │  │  Block 7:         │
│  Trade Logs &     │  │  Plotly Charts    │
│  Performance      │  │  (Interactive)    │
│  Summary          │  │                   │
└───────────────────┘  └───────────────────┘
```

### Execution Flow per Notebook Block

```
┌─────────────────────────────────────────────────────────────────────┐
│                      NOTEBOOK EXECUTION ORDER                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  [Block 1] ──>  API Authentication                                  │
│      │              - HMAC-SHA256 signature generation              │
│      │              - Delta Exchange credential verification        │
│      ▼                                                              │
│  [Block 2] ──>  Strategy Configuration                              │
│      │              - Trading mode (LONG/SHORT/BOTH)                │
│      │              - Entry/exit times (IST or UTC)                 │
│      │              - TP/SL parameters                              │
│      │              - Position sizing & risk                        │
│      ▼                                                              │
│  [Block 3] ──>  Technical Indicators                                │
│      │              - VWAP bands (Session/Week/Month)               │
│      │              - SMA (9, 21, 50)                              │
│      │              - RSI (14), Bollinger Bands (20), ATR (14)     │
│      │              - Entry/exit signal functions                   │
│      ▼                                                              │
│  [Block 4] ──>  Data Acquisition                                    │
│      │              - Fetch OHLCV from Delta Exchange API           │
│      │              - Cache to pickle (.pkl)                        │
│      │              - Load into pandas DataFrame                    │
│      ▼                                                              │
│  [Block 5] ──>  Backtrader Execution                                │
│      │              - Create Cerebro instance                       │
│      │              - Add data feed, strategy, analyzers            │
│      │              - Run backtest                                  │
│      │              - Collect trade records                         │
│      ▼                                                              │
│  [Block 6] ──>  Trade Logs & Performance Analysis                   │
│      │              - Detailed trade-by-trade log                   │
│      │              - Performance summary metrics                   │
│      │              - Exit reason breakdown                         │
│      │              - CSV export                                    │
│      ▼                                                              │
│  [Block 7] ──>  Interactive Visualizations                          │
│                   - Plotly candlestick chart                        │
│                   - Indicator overlays (VWAP, SMA, BB)              │
│                   - Buy/Sell signal markers                         │
│                   - Volume bars & RSI panel                         │
│                   - HTML export                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📓 Notebook Reference

### Available Notebooks

| Notebook | Strategy Type | Core Logic | Best For |
|---|---|---|---|
| `Day_Trading_Strategy.ipynb` | Time-Based Day Trading | Single entry at 10:30 IST, exit at TP/SL/EOD | Longer backtests (180 days), data diagnostics |
| `VWAP_Trading_Strategy.ipynb` | VWAP Mean Reversion | Price deviation from VWAP bands + RSI confirmation | Mean reversion, both long/short, shorter backtests |
| `Untitled-1.ipynb` | Multi-Entry Day Trading | Multiple entries (18:00/20:00/22:00 IST), points-based TP/SL | Scalping optimization, multi-entry strategies |

### Feature Comparison

| Feature | Day_Trading_Strategy | VWAP_Trading_Strategy | Untitled-1 |
|---|---|---|---|
| **Entry Signal** | Time-based (10:30 IST) | VWAP bands + RSI | Time-based (18/20/22 IST) |
| **Trading Mode** | LONG_ONLY | BOTH | LONG_ONLY |
| **Indicators for Entry** | None (time only) | VWAP, RSI(14) | None (time only) |
| **Indicators (Logged)** | SMA, RSI, BB, ATR | VWAP bands, RSI | SMA, RSI, BB, ATR |
| **TP Mode** | Points ($350) | Percentage (0.5%) | Points ($300) |
| **SL Mode** | Points ($10,000) | Percentage (1.5%) | Points ($800) |
| **Time Zone** | IST | UTC | IST |
| **Session Window** | Full day | 08:00–22:30 UTC | 18:00–23:58 IST |
| **Backtest Days** | 180 | 30 | 180 |
| **Scalping Analysis** | Yes (300-pt target) | No | Yes (300-pt target) |
| **Data Diagnostics** | Yes (gap analysis) | No | No |
| **Chart Candles** | 5,000 | 2,000 | 5,000 |
| **Output HTML** | `day_trading_chart.html` | `vwap_trading_chart.html` | `day_trading_chart.html` |

---

## 🔧 Block-by-Block Technical Reference

### Block 1: API Authentication

**File:** Lines 67–119 (Day_Trading_Strategy), Lines 23–113 (VWAP)

#### HMAC-SHA256 Signature Generation

```python
def generate_signature(secret, message):
    """Generate HMAC-SHA256 signature for Delta Exchange API authentication."""
    message = bytes(message, 'utf-8')
    secret = bytes(secret, 'utf-8')
    hash_obj = hmac.new(secret, message, hashlib.sha256)
    return hash_obj.hexdigest()
```

#### Authentication Flow

```
User provides API_KEY + API_SECRET
        │
        ▼
Create test message → "test_authentication"
        │
        ▼
Generate HMAC-SHA256 signature
        │
        ▼
Verify signature format (first 20 chars displayed)
        │
        ▼
Print verification banner
        │
        ▼
Ready for authenticated requests
```

#### Configuration Parameters

| Variable | Type | Description |
|---|---|---|
| `API_KEY` | `str` | Delta Exchange API key |
| `API_SECRET` | `str` | Delta Exchange API secret |
| `BASE_URL` | `str` | `https://api.delta.exchange` |
| `HISTORY_URL` | `str` | `https://api.india.delta.exchange/v2/history/candles` |

---

### Block 2: Strategy Configuration

**File:** Lines 187–318 (Day_Trading_Strategy), Lines 115–309 (VWAP)

#### Configuration Parameters Table

| Parameter | Type | Options / Range | Default | Description |
|---|---|---|---|---|
| `TRADING_MODE` | `str` | `LONG_ONLY`, `SHORT_ONLY`, `BOTH` | `LONG_ONLY` | Directional trading bias |
| `ENTRY_HOUR` | `int` | 0–23 | 10 | Entry hour (IST) |
| `ENTRY_MIN` | `int` | 0–59 | 30 | Entry minute (IST) |
| `EXIT_HOUR` | `int` | 0–23 | 23 | End-of-day exit hour |
| `EXIT_MIN` | `int` | 0–59 | 58 | End-of-day exit minute |
| `USE_TAKE_PROFIT` | `bool` | `True`, `False` | `True` | Enable take profit |
| `TP_MODE` | `str` | `PERCENTAGE`, `POINTS`, `BOTH` | `POINTS` | TP calculation mode |
| `TAKE_PROFIT_PERCENT` | `float` | 0.0–100.0 | 0.4 | TP as % of entry price |
| `TAKE_PROFIT_POINTS` | `float` | Any | 350 | TP in USD per trade |
| `USE_STOP_LOSS` | `bool` | `True`, `False` | `True` | Enable stop loss |
| `SL_MODE` | `str` | `PERCENTAGE`, `POINTS`, `BOTH` | `POINTS` | SL calculation mode |
| `STOP_LOSS_PERCENT` | `float` | 0.0–100.0 | 4.0 | SL as % of entry price |
| `STOP_LOSS_POINTS` | `float` | Any | 10000 | SL in USD per trade |
| `INITIAL_CAPITAL` | `float` | Any | 100000 | Starting capital in USD |
| `POSITION_SIZE` | `float` | Any | 1.0 | Quantity per trade (BTC) |
| `COMMISSION_PERCENT` | `float` | 0.0–100.0 | 0.0 | Brokerage commission % |
| `SLIPPAGE_USD` | `float` | Any | 0 | Slippage per trade in USD |
| `BACKTEST_DAYS` | `int` | 1–365 | 180 | Days of historical data |
| `TIMEFRAME` | `str` | `1min`, `5min`, `15min`, `1h` | `1min` | Candle resolution |
| `SYMBOL` | `str` | Any Delta pair | `BTCUSD` | Trading pair |

#### VWAP-Specific Parameters (VWAP_Trading_Strategy)

| Parameter | Type | Default | Description |
|---|---|---|---|
| `VWAP_ANCHOR` | `str` | `Session` | VWAP anchoring mode (Session/Week/Month) |
| `VWAP_STD_DEV_MULTIPLIER` | `float` | 1.8 | Confidence band width (standard deviations) |
| `LONG_DISTANCE_PERCENT` | `float` | 0.1 | Min deviation % from VWAP for long entry |
| `SHORT_DISTANCE_PERCENT` | `float` | 0.1 | Min deviation % from VWAP for short entry |
| `RSI_PERIOD` | `int` | 14 | RSI lookback period |
| `RSI_OVERSOLD` | `int` | 35 | RSI oversold threshold |
| `RSI_OVERBOUGHT` | `int` | 65 | RSI overbought threshold |
| `SESSION_START` | `str` | `08:00` | Trading session start (UTC) |
| `SESSION_END` | `str` | `22:30` | Trading session end (UTC) |
| `USE_CENTER_VWAP_EXIT` | `bool` | `True` | Exit when price reverts to VWAP |

---

### Block 3: Technical Indicators

**File:** Lines 357–436 (Day_Trading_Strategy), Lines 311–460 (VWAP)

#### Indicator Functions

| Function | Notebook | Returns | Description |
|---|---|---|---|
| `calculate_moving_averages(df, periods)` | Both | `DataFrame` | SMA for configurable periods (9, 21, 50) |
| `calculate_rsi(df, period)` | Both | `DataFrame` | Relative Strength Index (14) |
| `calculate_bollinger_bands(df, period)` | Both | `DataFrame` | Bollinger Bands (20, 2σ) |
| `calculate_atr(df, period)` | Both | `DataFrame` | Average True Range (14) |
| `calculate_vwap_bands(df)` | VWAP only | `DataFrame` | VWAP + confidence bands (stddev) |
| `check_session_time(timestamp)` | VWAP only | `bool` | Is within trading session? |
| `check_long_entry(row)` | VWAP only | `bool` | VWAP + RSI long signal |
| `check_short_entry(row)` | VWAP only | `bool` | VWAP + RSI short signal |
| `is_entry_time(timestamp)` | Day only | `bool` | Matches configured entry time |
| `is_exit_time(timestamp)` | Day only | `bool` | Matches configured exit time |

#### VWAP Calculation (VWAP_Trading_Strategy)

```python
def calculate_vwap_bands(df):
    df = df.copy()
    # Typical Price = (High + Low + Close) / 3
    df['tp'] = (df['high'] + df['low'] + df['close']) / 3
    # Cumulative VWAP
    df['cum_vwap'] = (df['tp'] * df['volume']).cumsum() / df['volume'].cumsum()
    # Standard deviation of typical price
    df['tp_std'] = df['tp'].rolling(window=len(df)).std()
    # Confidence bands
    df['upper_band'] = df['cum_vwap'] + (df['tp_std'] * VWAP_STD_DEV_MULTIPLIER)
    df['lower_band'] = df['cum_vwap'] - (df['tp_std'] * VWAP_STD_DEV_MULTIPLIER)
    return df
```

#### Entry Logic (VWAP Mean Reversion)

```
LONG ENTRY conditions:
    1. Current time is within trading session (08:00–22:30 UTC)
    2. Price < lower_band (price below -Nσ from VWAP)
    3. Price < VWAP × (1 - LONG_DISTANCE_PERCENT / 100)
    4. RSI < RSI_OVERSOLD (oversold condition)

SHORT ENTRY conditions:
    1. Current time is within trading session (08:00–22:30 UTC)
    2. Price > upper_band (price above +Nσ from VWAP)
    3. Price > VWAP × (1 + SHORT_DISTANCE_PERCENT / 100)
    4. RSI > RSI_OVERBOUGHT (overbought condition)
```

---

### Block 4: Data Acquisition

**File:** Lines 448–590 (Day_Trading_Strategy), Lines 462–643 (VWAP)

#### Data Fetching Pipeline

```python
def fetch_delta_data(symbol, days, timeframe):
    cache_file = f'{symbol}_{days}d_{timeframe}.pkl'

    # 1. Check cache first
    if os.path.exists(cache_file):
        with open(cache_file, 'rb') as f:
            return pickle.load(f)

    # 2. Fetch from API (batched by day for large ranges)
    all_candles = []
    for day_offset in range(days):
        start = (datetime.now() - timedelta(days=day_offset + 1)).isoformat()
        end = (datetime.now() - timedelta(days=day_offset)).isoformat()

        response = requests.get(HISTORY_URL, params={
            'symbol': symbol,
            'resolution': timeframe,
            'start': start,
            'end': end,
        }, headers=headers)

        candles = response.json().get('result', [])
        all_candles.extend(candles)

    # 3. Convert to DataFrame
    df = pd.DataFrame(all_candles)
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='s')

    # 4. Cache to disk
    with open(cache_file, 'wb') as f:
        pickle.dump(df, f)

    return df
```

#### Caching Strategy

- **Cache file naming:** `{symbol}_{days}d_{timeframe}.pkl`
- **First run:** Fetches from API, saves to `.pkl`
- **Subsequent runs:** Loads from `.pkl` (seconds vs. minutes)
- **To refresh:** Delete the `.pkl` file
- **Storage:** ~50–100 MB per 180 days of 1-min BTCUSD data

#### API Request Details

| Parameter | Value |
|---|---|
| **Endpoint** | `https://api.india.delta.exchange/v2/history/candles` |
| **Method** | `GET` |
| **Auth** | HMAC-SHA256 signature in headers |
| **Rate Limit** | 10 requests per second |
| **Batching** | 1 day per request (180 requests for 180 days) |

---

### Block 5: Backtrader Engine

**File:** Lines 801–1101 (Day_Trading_Strategy), Lines 722–922 (VWAP)

#### Strategy Class Architecture

```python
class DayTradingStrategy(bt.Strategy):
    """Event-driven trading strategy for backtrader."""

    params = (
        ('entry_time', ENTRY_TIME_IN_MIN),
        ('exit_time', EXIT_TIME_IN_MIN),
        ('long_enabled', LONG_ENABLED),
        ('short_enabled', SHORT_ENABLED),
        ('use_tp', USE_TAKE_PROFIT),
        ('tp_mode', TP_MODE),
        ('tp_percent', TAKE_PROFIT_PERCENT),
        ('tp_points', TAKE_PROFIT_POINTS),
        ('use_sl', USE_STOP_LOSS),
        ('sl_mode', SL_MODE),
        ('sl_percent', STOP_LOSS_PERCENT),
        ('sl_points', STOP_LOSS_POINTS),
    )

    def __init__(self):
        # Initialize indicators
        self.sma9 = bt.indicators.SimpleMovingAverage(self.data.close, period=9)
        self.sma21 = bt.indicators.SimpleMovingAverage(self.data.close, period=21)
        self.sma50 = bt.indicators.SimpleMovingAverage(self.data.close, period=50)
        self.rsi = bt.indicators.RSI(self.data.close, period=14)
        self.bb = bt.indicators.BollingerBands(self.data.close, period=20)

        # Trade tracking
        self.entry_price = None
        self.entry_time = None
        self.trades_count = 0
        self.last_entry_day = None

    def next(self):
        """Called for each new bar in the data feed."""
        current_time = self.data.datetime.datetime(0)
        current_minutes = current_time.hour * 60 + current_time.minute

        # ---- EXIT LOGIC ----
        if self.position:
            pnl_pct = (self.data.close[0] - self.entry_price) / self.entry_price * 100

            # Take Profit check
            if self.p.use_tp:
                if self.p.tp_mode in ('PERCENTAGE', 'BOTH'):
                    if pnl_pct >= self.p.tp_percent:
                        self.close()
                        self._record_trade(current_time, 'TAKE_PROFIT')
                        return

                if self.p.tp_mode in ('POINTS', 'BOTH'):
                    tp_price = self.entry_price + (self.p.tp_points / self.p.size)
                    if self.data.close[0] >= tp_price:
                        self.close()
                        self._record_trade(current_time, 'TAKE_PROFIT')
                        return

            # Stop Loss check
            if self.p.use_sl:
                if self.p.sl_mode in ('PERCENTAGE', 'BOTH'):
                    if pnl_pct <= -self.p.sl_percent:
                        self.close()
                        self._record_trade(current_time, 'STOP_LOSS')
                        return

                if self.p.sl_mode in ('POINTS', 'BOTH'):
                    sl_price = self.entry_price - (self.p.sl_points / self.p.size)
                    if self.data.close[0] <= sl_price:
                        self.close()
                        self._record_trade(current_time, 'STOP_LOSS')
                        return

            # End of Day exit
            if current_minutes >= self.p.exit_time:
                self.close()
                self._record_trade(current_time, 'END_OF_DAY')
                return

        # ---- ENTRY LOGIC ----
        if not self.position:
            today = current_time.toordinal()
            if self.last_entry_day != today:
                if current_minutes >= self.p.entry_time:
                    if self.p.long_enabled:
                        self.buy()
                        self.entry_price = self.data.close[0]
                        self.last_entry_day = today
```

#### VWAP Strategy Class (VWAP_Trading_Strategy)

```python
class VWAPMeanReversionStrategy(bt.Strategy):
    """Mean reversion strategy using VWAP bands and RSI."""

    params = (
        ('vwap_stdev', 1.8),
        ('long_enabled', True),
        ('short_enabled', True),
        ('long_distance', 0.1),
        ('short_distance', 0.1),
        ('use_vwap_exit', True),
        ('use_tp', True),
        ('tp_percent', 0.5),
        ('use_sl', True),
        ('sl_percent', 1.5),
        ('session_start', '08:00'),
        ('session_end', '22:30'),
        ('rsi_period', 14),
    )

    def __init__(self):
        tp = (self.data.high + self.data.low + self.data.close) / 3.0
        self.vwap = bt.indicators.SimpleMovingAverage(tp, period=20)
        self.rsi = bt.indicators.RSI(self.data.close, period=self.p.rsi_period)
        self.stdev = bt.indicators.StdDev(self.data.close, period=20)

    def next(self):
        # Entry: LONG when price < VWAP - Nσ AND RSI < oversold
        # Entry: SHORT when price > VWAP + Nσ AND RSI > overbought
        # Exit: VWAP center reversion, TP, SL, session end
```

#### Cerebro Setup

```python
cerebro = bt.Cerebro()

# Add data feed
data_feed = bt.feeds.PandasData(dataname=df)
cerebro.adddata(data_feed)

# Add strategy
cerebro.addstrategy(DayTradingStrategy)

# Add analyzers
cerebro.addanalyzer(bt.analyzers.SharpeRatio, _name='sharpe')
cerebro.addanalyzer(bt.analyzers.DrawDown, _name='drawdown')
cerebro.addanalyzer(bt.analyzers.Returns, _name='returns')
cerebro.addanalyzer(bt.analyzers.TradeAnalyzer, _name='trades')

# Set initial capital
cerebro.broker.setcash(INITIAL_CAPITAL)

# Run backtest
results = cerebro.run()
strategy = results[0]
```

---

### Block 6: Trade Logs & Performance Analysis

**File:** Lines 1102–1680 (Day_Trading_Strategy), Lines 1012–1223 (VWAP)

#### Performance Summary Output

```
╔══════════════════════════════════════════════════════════════════╗
║                   PERFORMANCE SUMMARY                            ║
╠══════════════════════════════════════════════════════════════════╣
║  Total Return      : +12.45%                ($12,450.00)        ║
║  Win Rate          : 68.42%  (26 wins / 12 losses / 38 total)   ║
║  Profit Factor     : 2.31    (Gross Profit / Gross Loss)        ║
║  Total Trades      : 38                                         ║
║  Max Drawdown      : -4.21%  (-$4,210.00)                       ║
║  Sharpe Ratio      : 1.87                                       ║
║  Avg Trade Duration: 2h 34m                                     ║
║  Avg Win           : +$845.50                                    ║
║  Avg Loss          : -$365.20                                    ║
║  Best Trade        : +$2,450.00                                  ║
║  Worst Trade       : -$1,200.00                                  ║
║  Consecutive Wins  : 7                                           ║
║  Consecutive Losses: 3                                           ║
╚══════════════════════════════════════════════════════════════════╝
```

#### Exit Reason Breakdown

```
EXIT REASON BREAKDOWN
─────────────────────────────────────────────
  TAKE_PROFIT  : 22 trades (57.89%)  ✓
  STOP_LOSS    : 10 trades (26.32%)  ✗
  END_OF_DAY   :  6 trades (15.79%)  ⚡
─────────────────────────────────────────────
```

#### Trade Log Format

```
┌──────┬────────────┬────────────┬────────┬────────┬───────────┬──────────┬──────────────┐
│ Trade│ Entry Time │ Exit Time  │  P&L   │ Return │ Exit      │ Duration │ R:R          │
│  #   │ (IST)      │ (IST)      │  ($)   │  (%)   │ Reason    │          │ (Fav/Adv)    │
├──────┼────────────┼────────────┼────────┼────────┼───────────┼──────────┼──────────────┤
│  1   │ 10:30      │ 11:15      │ +450   │ +0.45% │ TP        │ 0h 45m   │ 2.5          │
│  2   │ 10:30      │ 10:48      │ -320   │ -0.32% │ SL        │ 0h 18m   │ 0.3          │
│  3   │ 10:30      │ 15:15      │ +180   │ +0.18% │ EOD       │ 4h 45m   │ 1.2          │
└──────┴────────────┴────────────┴────────┴────────┴───────────┴──────────┴──────────────┘
```

---

### Block 7: Interactive Visualizations

**File:** Lines 2336–2758 (Day_Trading_Strategy), Lines 1338–1571 (VWAP)

#### Plotly Figure Structure

```
┌────────────────────────────────────────────────────────────────┐
│  Row 1: Candlestick Chart (height: 60%)                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  OHLC Candles (go.Candlestick)                          │  │
│  │  ─── SMA 9  (go.Scatter, yellow)                       │  │
│  │  ─── SMA 21 (go.Scatter, orange)                       │  │
│  │  ─── SMA 50 (go.Scatter, red)                          │  │
│  │  ─── Bollinger Upper (go.Scatter, dash)                │  │
│  │  ─── Bollinger Lower (go.Scatter, dash, fill)          │  │
│  │  ▲ BUY Signals (go.Scatter, green triangle-up)         │  │
│  │  ▼ SELL Signals (go.Scatter, red triangle-down)        │  │
│  │  [1D] [1W] [2W] [1M] [3M] [All] ← Time Range Selector │  │
│  └─────────────────────────────────────────────────────────┘  │
├────────────────────────────────────────────────────────────────┤
│  Row 2: Volume (height: 20%)                                   │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  Volume bars (go.Bar, green ↑ / red ↓)                 │  │
│  └─────────────────────────────────────────────────────────┘  │
├────────────────────────────────────────────────────────────────┤
│  Row 3: RSI (height: 20%)                                      │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  RSI line (go.Scatter, orange)                          │  │
│  │  ─ ─ Overbought (70) reference line                     │  │
│  │  ─ ─ Oversold (30) reference line                       │  │
│  └─────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

#### VWAP Chart (VWAP_Trading_Strategy)

```
┌────────────────────────────────────────────────────────────────┐
│  Row 1: Candlestick Chart (height: 60%)                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  OHLC Candles                                          │  │
│  │  ─── VWAP (go.Scatter, blue, width=2)                  │  │
│  │  ─ ─ Upper Band (go.Scatter, red, dash)               │  │
│  │  ─ ─ Lower Band (go.Scatter, green, dash)             │  │
│  │  ▲ LONG Entry (go.Scatter, green triangle-up)          │  │
│  │  ▼ SHORT Entry (go.Scatter, red triangle-down)         │  │
│  └─────────────────────────────────────────────────────────┘  │
├────────────────────────────────────────────────────────────────┤
│  Row 2: Volume (height: 20%)                                   │
├────────────────────────────────────────────────────────────────┤
│  Row 3: RSI (height: 20%)                                      │
│  │  ─ ─ Overbought (70) / Oversold (30)                       │
└────────────────────────────────────────────────────────────────┘
```

---

## 📚 API Reference

### Core Functions

#### `generate_signature(secret, message)`
- **Purpose:** Creates HMAC-SHA256 signature for API authentication
- **Inputs:**
  - `secret` (`str`) — API secret key
  - `message` (`str`) — Message to sign
- **Returns:** `str` — Hex digest of HMAC-SHA256 hash
- **Location:** Block 1, all notebooks

#### `calculate_moving_averages(df, periods)`
- **Purpose:** Computes SMAs for given periods
- **Inputs:**
  - `df` (`DataFrame`) — OHLCV data
  - `periods` (`list[int]`) — List of SMA periods (default: `[9, 21, 50]`)
- **Returns:** `DataFrame` — Original data with `SMA_9`, `SMA_21`, `SMA_50` columns
- **Edge case:** Returns NaN for first `period-1` rows

#### `calculate_rsi(df, period)`
- **Purpose:** Computes Relative Strength Index
- **Inputs:**
  - `df` (`DataFrame`) — Must contain `close` column
  - `period` (`int`) — Lookback period (default: 14)
- **Returns:** `DataFrame` with `RSI` column (0–100 scale)
- **Edge case:** First `period+1` rows are NaN; if loss is 0, RSI = 100

#### `calculate_bollinger_bands(df, period)`
- **Purpose:** Computes Bollinger Bands (2 standard deviations)
- **Inputs:**
  - `df` (`DataFrame`) — Must contain `close` column
  - `period` (`int`) — Lookback period (default: 20)
- **Columns added:** `BB_MA`, `BB_STD`, `BB_UPPER`, `BB_LOWER`

#### `calculate_vwap_bands(df)`
- **Purpose:** Computes VWAP with confidence bands
- **Inputs:** `df` (`DataFrame`) — Must contain `high`, `low`, `close`, `volume`
- **Columns added:** `tp`, `cum_vwap`, `tp_std`, `upper_band`, `lower_band`
- **Anchoring:** Session / Week / Month (configurable)

#### `fetch_delta_data(symbol, days, timeframe)`
- **Purpose:** Fetches historical OHLCV data from Delta Exchange
- **Inputs:**
  - `symbol` (`str`) — Trading pair (e.g., `BTCUSD`)
  - `days` (`int`) — Days of history (default: 180)
  - `timeframe` (`str`) — Candle resolution (default: `1min`)
- **Returns:** `DataFrame` with `timestamp`, `open`, `high`, `low`, `close`, `volume`
- **Caching:** Auto-saves/loads from `{symbol}_{days}d_{timeframe}.pkl`

### Backtraper Integration Functions

#### `run_backtest(df, strategy_class)`
- **Purpose:** Configures and executes backtrader backtest
- **Inputs:**
  - `df` (`DataFrame`) — OHLCV data
  - `strategy_class` (`bt.Strategy`) — Strategy class to test
- **Returns:** `tuple(strategy, cerebro)` — Results and engine instance

#### `analyze_performance(strategy)`
- **Purpose:** Extracts performance metrics from backtest results
- **Returns:** `dict` with keys: `total_return`, `win_rate`, `profit_factor`, `max_drawdown`, `sharpe_ratio`, `total_trades`, `avg_win`, `avg_loss`

---

## ⚙️ Strategy Configuration Reference

### Trading Mode Selection

```python
TRADING_MODE = "LONG_ONLY"   # Long positions only
TRADING_MODE = "SHORT_ONLY"  # Short positions only
TRADING_MODE = "BOTH"        # Both directions
```

### Position Sizing

```python
POSITION_SIZE = 1.0    # 1 BTC per trade (fixed)
# Future: percentage-based sizing:
# POSITION_SIZE_PERCENT = 10  # 10% of capital per trade
```

### TP/SL Configuration Modes

```
MODE: PERCENTAGE
  TP: entry_price × (1 + TAKE_PROFIT_PERCENT / 100)
  SL: entry_price × (1 - STOP_LOSS_PERCENT / 100)

MODE: POINTS
  TP: entry_price + (TAKE_PROFIT_POINTS / POSITION_SIZE)
  SL: entry_price - (STOP_LOSS_POINTS / POSITION_SIZE)

MODE: BOTH
  Triggers when EITHER condition is met (whichever comes first)
```

### Time Configuration

| Variable | Format | Example | Description |
|---|---|---|---|
| `ENTRY_HOUR` | `int` (0–23) | `10` | Entry hour (IST) |
| `ENTRY_MIN` | `int` (0–59) | `30` | Entry minute (IST) |
| `EXIT_HOUR` | `int` (0–23) | `23` | EOD exit hour (IST) |
| `EXIT_MIN` | `int` (0–59) | `58` | EOD exit minute (IST) |
| `ENTRY_TIME_IN_MIN` | computed | `630` | `ENTRY_HOUR × 60 + ENTRY_MIN` |
| `EXIT_TIME_IN_MIN` | computed | `1438` | `EXIT_HOUR × 60 + EXIT_MIN` |

---

## 📊 Technical Indicators Library

### Included Indicators

| Indicator | Default Period | Formula | Usage |
|---|---|---|---|
| **SMA** | 9, 21, 50 | `Σ(price) / N` | Trend direction |
| **RSI** | 14 | `100 - (100 / (1 + RS))` | Overbought/oversold |
| **Bollinger Bands** | 20, 2σ | `MA ± (σ × multiplier)` | Volatility & extremes |
| **ATR** | 14 | `EMA(TR, N)` | Volatility measurement |
| **VWAP** | Session | `Σ(TP × V) / Σ(V)` | Fair value anchor |

### TA-Lib vs NumPy Fallback

When TA-Lib is installed, indicator calculations are significantly faster:

| Indicator | TA-Lib | NumPy Fallback |
|---|---|---|
| SMA | `talib.SMA(df['close'], period)` | `df['close'].rolling(period).mean()` |
| RSI | `talib.RSI(df['close'], period)` | Manual calculation via gain/loss |
| Bollinger Bands | `talib.BBANDS(df['close'], period)` | Manual `rolling().mean()` + `rolling().std()` |
| ATR | `talib.ATR(high, low, close, period)` | Manual TR → EMA calculation |

---

## 🚀 Backtrader Engine Deep Dive

### Event Loop Lifecycle

```
backtrader Event Loop (for each bar):
│
├─ 1. Pre-next()
│     └─ Update broker cash & positions
│
├─ 2. next() called on Strategy
│     ├─ Update indicators (auto-computed)
│     ├─ Check exit conditions
│     │   ├─ Take Profit hit? → Close position
│     │   ├─ Stop Loss hit? → Close position
│     │   └─ EOD reached? → Close position
│     ├─ Check entry conditions
│     │   ├─ Entry time matched?
│     │   ├─ Not already in position?
│     │   └─ Not entered today? → Enter position
│     └─ Record trade stats
│
├─ 3. Post-next()
│     └─ Update analyzer metrics
│
└─ 4. Notify(order)
      └─ Log order execution details
```

### Order Execution Types

| Order Type | When Used | Description |
|---|---|---|
| `self.buy()` | Entry signal | Market order (current close price) |
| `self.close()` | Exit signal | Market order to flatten position |

### Analyzers

| Analyzer | Metrics Provided |
|---|---|
| `bt.analyzers.SharpeRatio` | Sharpe ratio (risk-adjusted return) |
| `bt.analyzers.DrawDown` | Max drawdown %, max drawdown value, duration |
| `bt.analyzers.Returns` | Total return %, annualized return |
| `bt.analyzers.TradeAnalyzer` | Win/loss count, avg win/loss, profit factor, consecutive wins/losses |

---

## 📥 Data Pipeline

### Data Format

| Column | Type | Example | Description |
|---|---|---|---|
| `timestamp` | `datetime` | `2026-01-15 10:30:00` | Candle open time |
| `open` | `float` | `67432.50` | Open price |
| `high` | `float` | `67890.00` | High price |
| `low` | `float` | `67210.00` | Low price |
| `close` | `float` | `67750.25` | Close price |
| `volume` | `float` | `125.43` | Volume in base currency |

### Caching Flow

```
┌─────────────────┐
│  Start Fetch    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     YES     ┌─────────────────┐
│   Cache file    │────────────>│   Load from     │
│   exists?       │             │   pickle (.pkl) │
└────────┬────────┘             └─────────────────┘
         │ NO
         ▼
┌─────────────────┐
│  API Request    │
│  (batched)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Parse JSON →   │
│  DataFrame      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Save to .pkl   │
└─────────────────┘
```

---

## 📈 Performance Metrics & Trade Analysis

### Metrics Definitions

| Metric | Formula | Description |
|---|---|---|
| **Total Return** | `(final_value - initial_capital) / initial_capital × 100` | Overall strategy return |
| **Win Rate** | `winning_trades / total_trades × 100` | Percentage of profitable trades |
| **Profit Factor** | `gross_profit / gross_loss` | Ratio of winning to losing money |
| **Max Drawdown** | `max(peak - trough) / peak × 100` | Largest peak-to-trough decline |
| **Sharpe Ratio** | `(return - risk_free) / std_dev(returns)` | Risk-adjusted return |
| **R:R Ratio** | `avg_win / avg_loss` | Reward-to-risk ratio |

### Trade Tracking Per Trade

```python
trade_record = {
    'trade_number': 1,
    'entry_time': datetime(2026, 1, 15, 10, 30),
    'exit_time': datetime(2026, 1, 15, 11, 15),
    'entry_price': 67432.50,
    'exit_price': 67882.50,
    'pnl': 450.00,
    'pnl_percent': 0.45,
    'exit_reason': 'TAKE_PROFIT',       # One of: TP, SL, EOD
    'duration_minutes': 45,
    'favorable_move': 500.00,           # Max favorable excursion
    'adverse_move': 120.00,             # Max adverse excursion
    'r_r_ratio': 4.17,                 # favorable / adverse
}
```

---

## 📈 Plotly Visualization Guide

### Figure Configuration

```python
fig = make_subplots(
    rows=3, cols=1,
    shared_xaxes=True,
    vertical_spacing=0.05,
    row_heights=[0.6, 0.2, 0.2],
    subplot_titles=('Price & Indicators', 'Volume', 'RSI (14)')
)
```

### Trace Types

| Trace | Type | Row | Color | Description |
|---|---|---|---|---|
| Candlesticks | `go.Candlestick` | 1 | Green/Red | OHLC price action |
| SMA 9 | `go.Scatter` | 1 | Yellow | Fast trend |
| SMA 21 | `go.Scatter` | 1 | Orange | Medium trend |
| SMA 50 | `go.Scatter` | 1 | Red | Slow trend |
| Bollinger Upper | `go.Scatter` | 1 | Gray (dash) | Upper volatility band |
| Bollinger Lower | `go.Scatter` | 1 | Gray (dash, fill) | Lower volatility band |
| BUY Signals | `go.Scatter` | 1 | Green (triangle-up) | Long entry markers |
| SELL Signals | `go.Scatter` | 1 | Red (triangle-down) | Short/exit markers |
| Volume | `go.Bar` | 2 | Green/Red | Trading volume |
| RSI | `go.Scatter` | 3 | Orange | Momentum oscillator |
| OB/OS Lines | `go.Scatter` | 3 | Gray (dash) | 70/30 reference lines |

### Interaction Features

- **Hover:** Unified hovermode shows all indicators at cursor position
- **Zoom:** Box select and pan tools in toolbar
- **Range Slider:** Date range selector at bottom
- **Range Buttons:** 1D, 1W, 2W, 1M, 3M, All — quick time period switching

### Export

```python
# Save as standalone HTML file
fig.write_html('day_trading_chart.html')

# Or as static image (requires kaleido)
fig.write_image('day_trading_chart.png')
```

---

## 🤖 Live Trading Automation

The backtesting framework is designed to be the **first step** toward a live trading bot. The same authentication, indicator, and signal logic can be reused.

### Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│    BACKTEST      │     │   PAPER TRADE    │     │   LIVE TRADE     │
│                  │     │                  │     │                  │
│  Historical      │     │  Real-time data  │     │  Real-time data  │
│  Data (past)     │     │  Simulated fills │     │  Real fills      │
│  No capital risk │     │  No capital risk │     │  Real capital    │
│  Framework:      │     │  Framework:      │     │  Framework:      │
│  backtrader      │     │  Custom script   │     │  Custom script   │
└──────────────────┘     └──────────────────┘     └──────────────────┘
        │                        │                        │
        ▼                        ▼                        ▼
All share same: API Auth (Block 1), Indicator Logic (Block 3),
                Signal Generation (Block 3), Config (Block 2)
```

### Automation Script Template

```python
# === LIVE TRADING BOT TEMPLATE ===
import time
import requests
import pandas as pd
from datetime import datetime

# Reuse Block 1 functions
API_KEY = "your_api_key"
API_SECRET = "your_api_secret"
BASE_URL = "https://api.delta.exchange"

def generate_signature(secret, message):
    import hashlib, hmac
    return hmac.new(bytes(secret, 'utf-8'),
                    bytes(message, 'utf-8'),
                    hashlib.sha256).hexdigest()

def get_latest_candle(symbol):
    """Fetch the most recent 1-min candle."""
    # Delta Exchange API call
    pass

def check_signal(candle, indicators):
    """Reuse Block 3 signal logic."""
    # VWAP / RSI / SMA conditions
    pass

def place_order(symbol, side, quantity):
    """Place a live order on Delta Exchange."""
    timestamp = str(int(time.time() * 1000))
    signature_payload = f"{timestamp}GET/v2/orders/create"
    signature = generate_signature(API_SECRET, signature_payload)

    headers = {
        'api-key': API_KEY,
        'timestamp': timestamp,
        'signature': signature,
    }

    payload = {
        'symbol': symbol,
        'side': side,        # 'buy' or 'sell'
        'order_type': 'market',
        'quantity': quantity,
    }

    response = requests.post(f"{BASE_URL}/v2/orders/create",
                           json=payload, headers=headers)
    return response.json()

# Main loop
while True:
    candle = get_latest_candle("BTCUSD")
    signal = check_signal(candle, indicators)

    if signal == "BUY" and not in_position:
        order = place_order("BTCUSD", "buy", 1.0)
        in_position = True
    elif signal == "SELL" and in_position:
        order = place_order("BTCUSD", "sell", 1.0)
        in_position = False

    time.sleep(60)  # Wait for next candle
```

---

## ⚡ Scalping Optimization Analysis

Both `Day_Trading_Strategy.ipynb` and `Untitled-1.ipynb` include a **scalping optimization** section that analyzes the potential for capturing a 300-point daily profit target.

### Analysis Logic

```python
# For each trade, calculate:
# 1. Max favorable price movement after entry
# 2. Time taken to reach each target level
# 3. Number of trades that reached N points profit
# 4. Average time to reach target

targets = [100, 200, 300, 500, 1000]
for trade in all_trades:
    max_favorable = trade['max_price'] - trade['entry_price']
    for target in targets:
        if max_favorable >= target:
            reached_targets[target] += 1
            total_time[target] += trade['duration_minutes']
```

### Output

```
SCALPING OPTIMIZATION — 300-POINT DAILY TARGET
─────────────────────────────────────────────────────
  Target    Trades Hit    Hit Rate    Avg Time
  (pts)     (out of 38)   (%)         to Hit
─────────────────────────────────────────────────────
  100       30            78.95%      0h 32m
  200       22            57.89%      1h 15m
  300       15            39.47%      2h 10m  ← Primary target
  500        8            21.05%      3h 45m
  1000       3             7.89%      5h 30m
─────────────────────────────────────────────────────
  Best strategy: Take 300 pts as target → 39% hit rate in 2h 10m avg
```

---

## 🤝 Contributing Guidelines

### How to Contribute

1. **Fork** the repository.
2. **Create a feature branch:**
   ```bash
   git checkout -b feature/new-strategy
   ```
3. **Add your strategy** as a new `.ipynb` notebook following the 7-block structure.
4. **Test** with at least 90 days of historical data.
5. **Commit:**
   ```bash
   git commit -m "feat: add [Strategy Name] strategy"
   ```
6. **Push:**
   ```bash
   git push origin feature/new-strategy
   ```
7. **Open a Pull Request.**

### Coding Standards

- **Notebook structure:** All notebooks must follow the 7-block pattern
- **Documentation:** Each block must have a markdown header and docstring
- **Configuration:** All configurable parameters must be in Block 2
- **Indicators:** Provide TA-Lib and NumPy fallback implementations
- **Credentials:** Never hardcode API keys — use placeholder variables

### What to Contribute

- [ ] New strategies (grid trading, momentum, arbitrage)
- [ ] Additional exchanges (Binance, Bybit, Coinbase)
- [ ] Machine learning signal generation (Scikit-Learn, XGBoost)
- [ ] Portfolio-level multi-asset backtesting
- [ ] Telegram/Discord alert integration
- [ ] Docker containerization
- [ ] Web dashboard (Streamlit, Gradio)
- [ ] Real-time streaming data via WebSocket

---

## 📜 Code of Conduct

### Our Pledge

We pledge to make participation in this project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity, experience level, nationality, personal appearance, race, religion, or sexual identity.

### Standards

**Positive behavior:**
- Using welcoming and inclusive language
- Respecting differing viewpoints
- Accepting constructive criticism gracefully
- Focusing on what is best for the community
- Showing empathy

**Unacceptable behavior:**
- Sexualized language or imagery
- Trolling, insults, or personal attacks
- Public or private harassment
- Publishing others' private information

### Enforcement

Report violations to [vijaykumar2572003@gmail.com](mailto:vijaykumar2572003@gmail.com). All complaints will be reviewed and investigated.

---

## 🔒 Security Policy

### Reporting Vulnerabilities

Email [vijaykumar2572003@gmail.com](mailto:vijaykumar2572003@gmail.com) with:
- Description of the vulnerability
- Steps to reproduce
- Potential impact

Response expected within 48 hours.

### Security Best Practices

| Practice | Recommendation |
|---|---|
| **API Keys** | Never commit real credentials. Use environment variables. |
| **API Secret** | Treat as a password — never share or hardcode. |
| **Trading Capital** | Start with paper trading before using real funds. |
| **Rate Limits** | Respect Delta Exchange API rate limits (10 req/s). |
| **Data Privacy** | No user data is collected or transmitted. |

---

## 📝 Changelog

### [1.0.0] — June 2026

#### Added
- 🚀 Three strategy notebooks: Day Trading, VWAP Mean Reversion, Multi-Entry
- 🔐 HMAC-SHA256 Delta Exchange API authentication
- ⚙️ Full strategy configuration system (mode, times, TP/SL, sizing)
- 📊 Technical indicators: SMA, RSI, Bollinger Bands, ATR, VWAP bands
- 📥 Automated data acquisition with pickle caching
- 🏎️ `backtrader` event-driven backtesting engine
- 📈 Interactive Plotly charts with 3-panel layout
- 📜 TradingView-style trade logs with exit reason analysis
- ⚡ Scalping optimization analysis for daily profit targets
- 📉 Performance metrics: Sharpe ratio, drawdown, profit factor
- 💰 Realistic cost modeling (commission, slippage)

---

## ❓ FAQ

**Q: Do I need a Delta Exchange account?**
A: Yes, you need API credentials from Delta Exchange. You can create a free account.

**Q: Can I use this with other exchanges?**
A: Currently Delta Exchange only, but the framework can be adapted. Contributions welcome!

**Q: How much data can I fetch?**
A: Up to 180 days of 1-minute data (or more with larger batch sizes).

**Q: How long does a backtest take?**
A: 180 days of 1-minute data (~259k candles) runs in 30–60 seconds on a modern laptop.

**Q: Can I run this in the cloud?**
A: Yes. The notebooks run anywhere Jupyter is available (Google Colab, AWS SageMaker, etc.).

**Q: Does the framework guarantee profitable strategies?**
A: No. Past performance does not guarantee future results. Always paper trade first.

**Q: How do I add a new indicator?**
A: Add it in Block 3's indicator functions section, then reference it in the strategy class in Block 5.

---

## 🔧 Troubleshooting Guide

### API Authentication Errors

| Error | Likely Cause | Solution |
|---|---|---|
| `406 Not Acceptable` | Invalid API key/secret | Verify credentials in Delta Exchange dashboard |
| `403 Forbidden` | IP not whitelisted | Add your IP to Delta API whitelist |
| `401 Unauthorized` | Signature mismatch | Check system clock (must be accurate to ±30s) |

### Data Fetching Issues

| Issue | Likely Cause | Solution |
|---|---|---|
| Empty DataFrame | No trades for symbol/timeframe | Verify symbol exists, try different pair |
| Missing days | API rate limit hit | Increase delay between requests |
| Stale data | Cache not refreshed | Delete `.pkl` files and re-run |

### Backtest Errors

| Error | Likely Cause | Solution |
|---|---|---|
| `ValueError: cannot convert float NaN to integer` | Insufficient warmup bars | Increase backtest days or reduce indicator periods |
| No trades executed | Entry conditions never met | Check entry time is within data range; verify symbol price |
| `KeyError: 'timestamp'` | Data format mismatch | Ensure Delta API output matches expected schema |

### Plotly Chart Issues

| Issue | Likely Cause | Solution |
|---|---|---|
| Blank chart | Data not passed correctly | Verify `chart_data` DataFrame is populated |
| `ImportError: No module named 'plotly'` | Missing dependency | `pip install plotly` |
| Slow rendering | Too many candles | Reduce `tail_candles` parameter (e.g., 2000 instead of 5000) |

---

## ⚠️ Disclaimer

> **IMPORTANT DISCLAIMER**
>
> **Trading cryptocurrency carries significant financial risk.**
>
> 1. **Past Performance:** Historical backtest results do not guarantee future performance. Markets change, and strategies that worked in the past may fail in the future.
>
> 2. **No Financial Advice:** This software is provided for educational and research purposes only. The authors are not financial advisors.
>
> 3. **Paper Trade First:** Always validate strategies in a paper/simulated environment before risking real capital.
>
> 4. **Risk Management:** Never trade more than you can afford to lose. Use appropriate position sizing and stop losses.
>
> 5. **No Liability:** The authors accept no liability for any financial losses incurred through the use of this software.
>
> 6. **Data Accuracy:** Market data from API sources may contain errors or gaps. Always verify critical findings with multiple data sources.

---

<p align="center">
  <b>Delta Crypto TradingView-Style Backtesting & Automation Framework</b>
  <br>
  <i>Backtest like a pro — without paying for Premium.</i>
  <br><br>
  <a href="https://github.com/vijaykumarGK-Developer/delta-crypto-trading-view-backtest-stratergy">GitHub</a> •
  <a href="https://www.delta.exchange/">Delta Exchange</a> •
  <a href="https://github.com/mementum/backtrader">backtrader</a> •
  <a href="https://plotly.com/">Plotly</a>
  <br><br>
  <sub>Built with Python, backtrader, Pandas, Plotly, and TA-Lib</sub>
</p>
