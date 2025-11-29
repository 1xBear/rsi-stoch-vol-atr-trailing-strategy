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

  ## Strategy Math (Formulas)

This section gives the core formulas behind the indicators used.

### Exponential Moving Average (EMA)

For a series of closes \( C_t \) and period \( N \):

- Smoothing factor:  
  \[
  \alpha = \frac{2}{N + 1}
  \]
- Recursive definition:
  \[
  \text{EMA}_t = \alpha C_t + (1 - \alpha)\text{EMA}_{t-1}
  \]

The strategy uses an EMA of length \( N = \text{emaLen} \) as the trend filter.

---

### Relative Strength Index (RSI)

For period \( N \):

1. Compute up-moves and down-moves:
   - \( U_t = \max(C_t - C_{t-1}, 0) \)
   - \( D_t = \max(C_{t-1} - C_t, 0) \)

2. Compute smoothed averages (usually Wilder’s smoothing or an EMA):
   - \( \text{AvgGain}_t \)
   - \( \text{AvgLoss}_t \)

3. Relative strength:
   \[
   RS_t = \frac{\text{AvgGain}_t}{\text{AvgLoss}_t}
   \]

4. RSI:
   \[
   \text{RSI}_t = 100 - \frac{100}{1 + RS_t}
   \]

The strategy only allows entries when:
\[
\text{RSI}_t < \text{rsiBuyLevel}
\]

---

### Stochastic Oscillator (%K)

For period \( N = \text{stochLen} \):

- Highest high:
  \[
  H^{(N)}_t = \max(H_{t-N+1}, \dots, H_t)
  \]
- Lowest low:
  \[
  L^{(N)}_t = \min(L_{t-N+1}, \dots, L_t)
  \]

Raw %K:
\[
\%K_{\text{raw},t} = 100 \cdot \frac{C_t - L^{(N)}_t}{H^{(N)}_t - L^{(N)}_t}
\]

Smoothed %K (what the script uses as \( k \)):
\[
k_t = \text{SMA}\big(\%K_{\text{raw}}, \text{smoothK}\big)
\]

Entry trigger is a crossover:
\[
\text{crossover}(k_t, \text{stochBuyLevel})
\]
meaning \( k_t \) crosses upward through the level.

---

### Average True Range (ATR)

True Range for each bar \( t \):

\[
\text{TR}_t = \max
\left(
  H_t - L_t,\,
  |H_t - C_{t-1}|,\,
  |L_t - C_{t-1}|
\right)
\]

ATR over period \( N = \text{atrLen} \) is typically an EMA of \(\text{TR}_t\):

\[
\text{ATR}_t = \text{EMA}_N(\text{TR}_t)
\]

---

### ATR-Based Trailing Stop

For a long position, the raw candidate stop is:

\[
\text{StopCandidate}_t = C_t - \text{ATR}_t \cdot M
\]

where \( M = \text{atrMultiplier} \) (e.g. \( M = 3.0 \)).

The trailing stop is a “ratchet”:

\[
\text{TrailStop}_t =
\begin{cases}
\text{StopCandidate}_t & \text{if no prior stop} \\
\max(\text{TrailStop}_{t-1}, \text{StopCandidate}_t) & \text{otherwise}
\end{cases}
\]

The exit condition is:

\[
\text{Exit when } C_t \leq \text{TrailStop}_t
\]
