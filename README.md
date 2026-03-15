# Options Pricing

A progressive series of option pricing models implemented in Python/Jupyter, applied to live **NSE NIFTY 50** index options. Models range from the classical Black-Scholes formula to advanced stochastic volatility with jump diffusion.

---

## Notebooks

| # | Notebook | Model | Complexity |
|---|----------|-------|------------|
| 01 | [01_black_scholes.ipynb](01_black_scholes.ipynb) | **Black-Scholes** | Closed-form analytical |
| 02 | [02_black_scholes_merton.ipynb](02_black_scholes_merton.ipynb) | **Black-Scholes-Merton** | Adds continuous dividend yield |
| 03 | [03_monte_carlo_simulation.ipynb](03_monte_carlo_simulation.ipynb) | **Monte Carlo (GBM)** | Numerical simulation + variance reduction |
| 04 | [04_heston_model.ipynb](04_heston_model.ipynb) | **Heston Stochastic Volatility** | MCS + Characteristic Function |
| 05 | [05_bates_model.ipynb](05_bates_model.ipynb) | **Bates (Heston + Jumps)** | MCS + Characteristic Function |
| 06 | [06_merton_jump_diffusion.ipynb](06_merton_jump_diffusion.ipynb) | **Merton Jump Diffusion** | Analytical series + calibration |

Each notebook is self-contained and Colab-compatible. The progression follows the natural evolution of option pricing theory.

---

## Models at a Glance

### 01 — Black-Scholes
The foundational closed-form model (1973). Assumes constant volatility, no dividends, European-style options.
- Prices calls and puts analytically
- Computes all Greeks (Delta, Gamma, Vega, Theta, Rho)
- Extracts implied volatility via Newton-Raphson
- Compares theoretical vs market prices for NIFTY options

### 02 — Black-Scholes-Merton
Extends Black-Scholes with a **continuous dividend yield** `q = 1.24%` (relevant for index options where dividends are paid continuously).
- Same structure as BS but all formulas adjusted for `e^{-qT}` discount

### 03 — Monte Carlo Simulation
Prices options by simulating **Geometric Brownian Motion** paths.
- Standard Monte Carlo + **Antithetic Variate** variance reduction
- Convergence analysis across `M = [10 … 10,000]` paths
- Demonstrates how pricing error scales with path count

### 04 — Heston Stochastic Volatility
Replaces constant σ with a mean-reverting **CIR variance process** (1993).
- Monte Carlo simulation of coupled SDEs
- Semi-analytical pricing via **Characteristic Function / Fourier Transform**
- Calibration to market prices via `scipy.optimize.minimize`
- Captures the volatility smile/skew missing from Black-Scholes
- Optional: BANKNIFTY calibration

### 05 — Bates Model
Combines **Heston stochastic volatility** with **Merton-style Poisson jumps** (1996).
- Full truncation scheme to ensure variance positivity
- Characteristic function is product: `φ_Bates = φ_Heston × φ_Jump`
- Captures the pronounced volatility skew seen in index options
- Better fits OTM options than Heston alone

### 06 — Merton Jump Diffusion
Extends Black-Scholes with **discrete random jumps** modelled as a Poisson process (1979).
- Closed-form series solution (sum over `k = 0…40` terms)
- Calibration via least-squares optimisation over `[σ, m, v, λ]`
- Applied to live NSE NIFTY data

---

## Data Source

All models use **live NSE NIFTY 50 option chain data** fetched directly from the NSE India public API.

```python
TARGET_EXPIRY = "27-Mar-2025"   # Update to a valid future monthly expiry
r = 0.06857                      # 91-day T-bill yield (6.857%)
q = 0.0124                       # Dividend yield — notebooks 02 only
# S0: fetched live from NSE API
# sigma: computed from 1-year NIFTY historical returns via yfinance
```

> **Before running:** Update `TARGET_EXPIRY` in each notebook to a valid future NSE monthly expiry date.

---

## Setup

### Local (Jupyter)
```bash
pip install numpy scipy matplotlib seaborn pandas yfinance requests plotly
jupyter notebook
```

### Google Colab
Open any notebook directly in Colab — all dependencies are available. Each notebook is fully self-contained.

---

## Project Structure

```
Options-Pricing/
├── 01_black_scholes.ipynb
├── 02_black_scholes_merton.ipynb
├── 03_monte_carlo_simulation.ipynb
├── 04_heston_model.ipynb           ← canonical NSE fetch utilities
├── 05_bates_model.ipynb
├── 06_merton_jump_diffusion.ipynb
├── README.md
└── CLAUDE.md                       ← project guide and progress log
```

### Branch Structure

| Branch | Contents |
|--------|----------|
| `unified` | All 6 notebooks standardised — **active** |
| `explore` | Original BS, MCS, Heston, Bates |
| `main` | Original BSM, Merton |

---

## Model Comparison

| Feature | BS | BSM | MCS | Heston | Bates | Merton JD |
|---------|:--:|:---:|:---:|:------:|:-----:|:---------:|
| Constant volatility | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ |
| Stochastic volatility | ✗ | ✗ | ✗ | ✓ | ✓ | ✗ |
| Price jumps | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ |
| Dividend yield | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Volatility smile | ✗ | ✗ | ✗ | ✓ | ✓ | ✓ |
| Calibration | IV only | IV only | ✗ | ✓ | ✓ | ✓ |
| Analytical formula | ✓ | ✓ | ✗ | CF | CF | ✓ |
