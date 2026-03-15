# CLAUDE.md — Options Pricing Project Guide

## Project Overview

Six Jupyter notebooks implementing progressively sophisticated option pricing models,
all applied to live NSE NIFTY 50 index options. Each notebook is self-contained and
Colab-compatible. Originally a collaborative project; unified into this branch.

---

## Notebook Progression

| # | File | Model | Key Feature |
|---|------|-------|-------------|
| 01 | `01_black_scholes.ipynb` | Black-Scholes | Closed-form, Greeks, IV via Newton-Raphson |
| 02 | `02_black_scholes_merton.ipynb` | Black-Scholes-Merton | Adds continuous dividend yield q |
| 03 | `03_monte_carlo_simulation.ipynb` | Monte Carlo (GBM) | Antithetic variance reduction |
| 04 | `04_heston_model.ipynb` | Heston (MCS + CF) | Stochastic volatility, calibration |
| 05 | `05_bates_model.ipynb` | Bates (MCS + CF) | Heston + Poisson jumps |
| 06 | `06_merton_jump_diffusion.ipynb` | Merton Jump Diffusion | Analytical series + calibration |

---

## Common Parameters (all notebooks)

```python
r = 0.06857          # Risk-free rate: 6.857% (91-day T-bill, India)
q = 0.0124           # Dividend yield: 1.24% — BSM notebook (02) ONLY
TARGET_EXPIRY = '...'  # Set to nearest valid monthly expiry before running

# Synthetic testing (identical across all notebooks):
S0_test   = 100.0
K_test    = 100.0
T_test    = 1.0
r_test    = 0.05
sigma_test = 0.20

# Heston/Bates synthetic:
v0_test = 0.04; kappa_test = 1.5; theta_test = 0.04
sigma_v_test = 0.3; rho_test = -0.7

# Bates additional: lambda_p_test=0.1, mu_J_test=-0.05, delta_test=0.1
# Merton additional: lam_test=0.75, m_test=0.0, v_jump_test=0.25
# Simulation:        N_steps=255, M_paths=10000
```

---

## Shared Utility Code

`fetch_nse_data(symbol, max_retries)` and `process_option_data()` /
`process_option_chain_data()` are **duplicated** in each notebook to preserve
Colab portability. **Canonical source: `04_heston_model.ipynb`.**

- Notebooks 01, 02, 03, 06 → `process_option_chain_data()` (returns `ltp_call`, `ltp_put`, `iv_call`, `iv_put`)
- Notebooks 04, 05 → `process_option_data()` (returns `price` bid-ask mid, `maturity`, `rate` — used for calibration)

**If you update the NSE fetch logic, update all 6 notebooks.**

NSE API notes:
- Requires three-step cookie warming: homepage → option chain page → API call
- Use `max_retries=3` with session refresh on failure
- `TARGET_EXPIRY` must be a valid future expiry date in `DD-MMM-YYYY` format (e.g. `'27-Mar-2025'`)

---

## Notebook Template Structure

Each notebook follows this section order (modeled on `04_heston_model.ipynb`):
1. Title & Mathematical Theory (markdown)
2. Imports (single consolidated code cell)
3. Model Implementation (class/function definitions)
4. Synthetic Parameter Testing (`S0_test=100`, etc.)
5. Simulated Path Visualization (where applicable)
6. Implied Volatility Smile (across strikes)
7. Real Market Data — NSE NIFTY (`fetch_nse_data` block)
8. Model Calibration (notebooks 04, 05, 06)
9. Summary (markdown)

---

## Branch Structure

| Branch | Contents |
|--------|----------|
| `main` | Original: `Black_Scholes_Merton_(3).ipynb`, `Merton_Jump_Diffusion_(1).ipynb` |
| `explore` | Original: `blackScholes.ipynb`, `mcs.ipynb`, `heston.ipynb`, `bates.ipynb` |
| `unified` | **All 6 notebooks standardized** (this branch — active) |
| `merton` | Mirrors explore |

---

## Dependencies

```
numpy, scipy, matplotlib, seaborn, requests, yfinance, pandas
```
Optional (Heston notebook): `py_vollib_vectorized`, `plotly`

---

## Progress Log

### 2026-03-15 — Initial Unification Complete
**Branch:** `unified` (created from `main`, explore notebooks cherry-picked in)

| Commit | Change |
|--------|--------|
| `618b7a2` | Brought all 6 notebooks into unified branch |
| `462c737` | Renamed all notebooks to `01_`–`06_` convention |
| `ba5da88` | Created `CLAUDE.md` |
| `c7e618d` | 01_black_scholes: consolidated imports, canonical NSE fetch, synthetic test, removed empty cells |
| `7e66cf8` | 02_black_scholes_merton: removed Colab badge, canonical NSE fetch, synthetic test |
| `4334e48` | 03_monte_carlo_simulation: consolidated imports, deduplicated NSE fetch, antithetic reuses fetched df |
| `d6fc511` | 04_heston_model: yfinance volatility, removed commented blocks, BANKNIFTY as optional section |
| `eaaa090` | 05_bates_model: canonical NSE fetch (adds max_retries), yfinance volatility |
| `78ddbe5` | 06_merton_jump_diffusion: replaced S&P500 CSV with NSE NIFTY calibration, standardized imports/params |
| `cfac3b7` | Final pass: TARGET_EXPIRY in heston, clarified synthetic r=0.02 |

**All 6 notebooks now:**
- Use canonical `fetch_nse_data()` with `max_retries=3`
- Compute `annual_volatility` dynamically via yfinance (no hardcoded 14.32)
- Use `TARGET_EXPIRY` variable (update before running)
- Have consolidated imports in top code cell
- Have no Colab badges or S&P500/external CSV dependencies
- Have synthetic parameter tests using `_test` suffix convention
