# RSI Stoch + Volume + ATR Trailing (TradingView Strategy)

A TradingView Pine Script strategy that combines:

- 200 EMA trend filter
- Volume above average confirmation
- RSI + Stochastic oscillator entries
- ATR-based trailing stop

Built for trending, volatile markets such as crypto, indices, and FX.

---

## Table of Contents

- [Overview](#overview)
- [Strategy Logic](#strategy-logic)
  - [Trend Filter](#trend-filter)
  - [Volume Filter](#volume-filter)
  - [Entry Conditions](#entry-conditions)
  - [Exit Conditions / Risk Management](#exit-conditions--risk-management)
- [Inputs](#inputs)
- [How to Use](#how-to-use)
- [Backtesting Notes](#backtesting-notes)
- [Repository Structure](#repository-structure)
- [Roadmap / Ideas](#roadmap--ideas)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This strategy looks for long entries in bullish conditions using a combination of:

- **Trend direction** from a long-term EMA
- **Volume confirmation** to avoid thin moves
- **RSI + Stochastic** for timing entries
- **ATR-based trailing stop** to adapt exits to volatility

Implemented in **Pine Script v6** as a `strategy` script.

---

## Strategy Logic

### Trend Filter

- Optional filter controlled by the `Use 200 EMA Trend Filter?` input.
- Uses an EMA of length `emaLen` (default: 200).
- Long entries are allowed only when:

  ```text
  close > emaTrend