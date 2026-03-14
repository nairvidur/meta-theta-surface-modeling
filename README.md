# META Options — Theta Decay Surface Model

A quantitative options analysis project modelling **Black-Scholes theta decay** across strike/moneyness and time dimensions for META short-dated options in the final 3 days before expiry (Jan 27–29, 2026, expiry Jan 30).

## Overview

This project builds a **3D theta surface** using real market data (bid/ask/IVM) scraped for META options across three consecutive trading days leading into a weekly expiry. The surface visualises how time decay accelerates as DTE → 0 and how it varies across moneyness.

## Key Features

- Parses real options chain data (calls & puts) from CSV exports
- Computes **Black-Scholes theta** ($/calendar day) for each strike
- Plots a **3D interpolated surface**: Days-to-Expiry × Log-Moneyness × Theta
- Cross-sectional slices by date showing the theta smile across moneyness
- Captures the **DeepSeek volatility spike** on Jan 28 (IVM ~98% vs ~78% on Jan 27)

## Methodology

| Parameter | Value |
|---|---|
| Underlying | META (Meta Platforms) |
| Expiry | January 30, 2026 |
| Data dates | Jan 27, 28, 29, 2026 |
| Risk-free rate | 4.25% (Fed Funds) |
| Theta convention | $/calendar day (annual ÷ 365) |
| Moneyness | Log-moneyness: ln(S/K) — ATM = 0 |
| Grid interpolation | `scipy.griddata` cubic |

## Results

### 3D Theta Surface
![Theta Surface](theta_surface_3d_moneyness.png)

### Cross-Sections by Date
![Cross Sections](theta_cross_sections_moneyness.png)

### Notable Observations

- **ATM theta peaks** in magnitude and accelerates sharply as DTE: 3 → 2 → 1
- **Jan 28 IVM spike (~98%)**: driven by the DeepSeek AI news event — theta is significantly elevated vs adjacent days
- **Volatility skew** is visible: puts carry higher IVM than calls on Jan 29, reflected in the asymmetric theta surface
- **OTM options** decay faster in relative terms near expiry — visible in the steep wings of the cross-sections

## Data Limitations

The options chain coverage is asymmetric across the three dates, due to META's spot price moving significantly (+$30 over 3 days) and liquidity drying up in far OTM strikes near expiry:

| Date | Spot | Strike Range | Coverage |
|---|---|---|---|
| Jan 27 | ~$672 | 640–702.5 | ✅ Balanced — strikes both below and above spot |
| Jan 28 | ~$693 | 637.5–700 | ⚠️ Skewed — almost all strikes below spot, few OTM calls |
| Jan 29 | ~$703 | 700–780 | ⚠️ Skewed — almost all strikes above spot, few OTM puts |

**Implications for the model:**
- On Jan 28 and Jan 29, `scipy.griddata` cubic interpolation fills in moneyness regions where no real quotes exist. These extrapolated areas should be interpreted with caution.
- The asymmetry on Jan 28 is partly explained by the DeepSeek-driven gap down — OTM calls likely went no-bid as the market moved away from them rapidly.
- Near-expiry far OTM options frequently have no meaningful market (wide spreads, zero volume), so the missing data reflects real market structure rather than a data collection error.

A future improvement would be to clip the surface to each day's valid moneyness range and visually mark interpolated regions.

## Setup

```bash
git clone https://github.com/nairvidur/meta-theta-surface
cd meta-theta-surface
pip install pandas numpy matplotlib scipy yfinance seaborn
jupyter notebook theta_surface_model_complete.ipynb
```

## File Structure

```
├── theta_surface_model_complete.ipynb   # Main notebook
├── 27-1.csv                             # Options chain Jan 27
├── 28-1.csv                             # Options chain Jan 28
├── 29-1.csv                             # Options chain Jan 29
├── theta_surface_3d_moneyness.png       # 3D surface output
├── theta_cross_sections_moneyness.png   # Cross-section output
└── README.md
```

## Dependencies

```
pandas, numpy, matplotlib, scipy, seaborn, yfinance
```

---
*Built with real market data. For educational and research purposes.*
