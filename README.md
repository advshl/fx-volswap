# FX Volatility Swap Overlay as a Gamma Hedge  
### EURUSD (2015–2024) — Volatility Risk Premium & Hedging Analysis

This project analyses how a **long-volatility overlay** (a vega-scaled proxy for a 3M volatility swap)  
can stabilise the P&L of a **short 1M ATM delta-hedged EURUSD straddle**.  
The objective is risk stabilisation rather than perfect replication, reflecting how FX desks manage  
short-gamma books in practice.

All steps — data preparation, volatility construction, P&L engines, and results — are contained in a **single reproducible notebook**.

---

## 🔍 1. Overview

A delta-hedged short straddle:

- earns **theta** when implied vol > realised vol  
- suffers **gamma losses** when spot volatility spikes  
- produces unstable P&L, especially around macro events (e.g., 2020)

To mitigate this, we introduce a **scaled long-vol overlay**.  
The overlay behaves like a 3M volatility swap, marked to market daily, with notional sized  
proportionally to the straddle’s vega using a hedge-intensity parameter **α**.

This mirrors risk-control overlays used by sell-side FX structuring and options desks.

---

## 📁 2. Repository Structure
```bash
fx-volswap-gamma-hedge
│
├── data
│ ├── eurusd_iv_data2.xlsx # Annual mean implied vols (1M & 3M)
│ ├── eurusd_vol_data2.csv # Spot, RV and synthetic IV prep data
│ ├── expanded_eurusd_vol_data.csv # Spot, RV and synthetic IV prep and calc data for cross-reference
│
├── FX_VolSwap_Project.ipynb # Full workflow notebook
│
├── requirements.txt
└── README.md
```

---

## 📊 3. Methodology

### 3.1 Synthetic Implied Volatility Construction  
Because long-term EURUSD ATM IV is not freely available, we construct IVs using:

- Rolling realised volatility (1M = 21d, 3M = 63d)
- Historical annual mean ATM implied vols (2015–2024)
- Volatility risk premia (≈1.5% for 1M, ≈1.4% for 3M)
- Exponential smoothing (30d for 1M, 60d for 3M)

This produces realistic, smooth IV series anchored to market-consistent levels.

---

### 3.2 Delta-Hedged Straddle P&L (Gamma–Theta Decomposition)

Daily P&L is:


$\text{PnL}_t = \text{Theta Carry} + \text{Gamma Loss}$


- **Theta / Variance Carry**

$$\text{PnL}_{\theta,t} = \frac{1}{2}(\sigma_{\text{imp}}^2 - \sigma_{\text{real}}^2)\frac{\text{vega}}{252}$$

- **Gamma Loss Approximation**

$$
\text{PnL}_{\gamma,t} =
-\frac{1}{2} \Gamma_{\$}(\Delta S/S)^2,
\quad
\Gamma_{\$} \approx \frac{\text{vega}}{\sigma_{\text{imp}} T}
$$

A daily scaling factor smooths gamma over the 1M maturity.

---

### 3.3 Volatility Swap Overlay (Proxy Hedge)

A 3M vol swap pays:


$\text{RV}_{3M} - K)\times\text{Notional}$.


Daily accrual is approximated as:

$$
\frac{(\text{RV}_{3M} - K)\times\text{Notional}}{63}.
$$

**Notional Scaling**

$$
\text{Notional} = \alpha \times \text{Vega}_{\text{total}} 
\times \text{Median}(\text{IV}_{1M})
$$

where **α** (0.2–0.8) controls hedge intensity.  
This is a **risk-scaled long-vol overlay**, not a strict market-quoted hedge.

---

### 3.4 Hedged Strategy Construction

$$
\text{PnL}_{\text{hedged}} =
\text{PnL}_{\text{straddle}} + \text{PnL}_{\text{swap}}.
$$

This improves stability by adding positive exposure to realised volatility.

---

## 📈 4. Results Summary

### Cumulative PnL  
- The **unhedged straddle** experiences gamma drawdowns and mild negative drift.  
- The **vol swap** is smooth and upward sloping (long RV premium).  
- The **hedged strategy** significantly reduces drawdowns and improves stability.

### Performance Metrics (Indicative)

| Strategy          | Sharpe | Ann. Vol | Max DD |
|------------------|--------|----------|--------|
| Short Straddle   | 0.30–0.40 | Moderate | High   |
| Vol Swap         | Positive | Low | Minimal |
| **Hedged**       | **1.5–3.0** | Moderate | **Lower** |

These figures are economically and structurally consistent with a long-vol overlay hedging a gamma-short profile.

---

## ⚙️ 5. How to Run

```bash
pip install -r requirements.txt
jupyter notebook FX_VolSwap_Project.ipynb

