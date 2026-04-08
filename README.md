# Ind AS 117 (IFRS 17) — Life Insurance CSM Model
### Indian Market | IALM 2012-14 | RBI G-Sec | Monte Carlo Risk Adjustment

**Author:** Prithiyanga Bintu Mani  
**Qualification:** MSc Actuarial Science — University of Kent (Merit, 2025)  
**IFoA Exemptions:** CM1, CM2, CB1, CB2, CS1, CS2  

---

## Overview

This project implements a **working actuarial model** under **Ind AS 117** (India's equivalent of IFRS 17) for a simulated portfolio of 1,000 Indian term life insurance policies.

India's insurance regulator IRDAI has mandated Ind AS 117 implementation by **FY2027 (April 2027)**. This model demonstrates the three core building blocks of the General Measurement Model (GMM):

| Component | Value | Description |
|-----------|-------|-------------|
| **BEL** (Best Estimate Liability) | ₹ −1.607 crore | EPV of future outflows minus inflows |
| **RA** (Risk Adjustment) | ₹ +0.997 crore | 75th percentile VaR via Monte Carlo |
| **CSM** (Contractual Service Margin) | ₹ +2.345 crore | Unearned profit released over 20 years |
| **Total Liability** | ₹ 1.735 crore | BEL + RA + CSM on Day 1 balance sheet |

---

## Data Sources

| Data | Source | URL |
|------|--------|-----|
| Mortality rates | IALM 2012-14 — Institute of Actuaries of India | [actuariesindia.org](https://www.actuariesindia.org/mortality-table) |
| Discount curve | CCIL Tenorwise G-Sec Yields — FY2025 | [ccilindia.com](https://www.ccilindia.com/tenorwise-indicative-yields) |
| Historical yields | RBI Database on Indian Economy | [data.rbi.org.in](https://data.rbi.org.in) |
| Expense benchmarks | IRDAI Annual Report 2024-25 | [irdai.gov.in](https://irdai.gov.in/annual-reports) |
| Regulatory standard | Ind AS 117 — MCA Gazette Aug 2024 | [mca.gov.in](https://www.mca.gov.in) |

---

## Project Structure

```
IndAS117-CSM-Model/
│
├── IndAS117_Model.py          ← Main Python model (run this)
├── dashboard.html             ← LinkedIn visual dashboard
├── Report_IndAS117.docx       ← Full actuarial report
│
├── outputs/
│   ├── Chart1_IALM_Mortality.png
│   ├── Chart2_Yield_Curve.png
│   ├── Chart3_CSM_Rollforward.png
│   ├── Chart4_MonteCarlo_BEL.png
│   ├── CSM_Rollforward.csv
│   └── Portfolio_Data.csv
│
└── README.md
```

---

## How to Run

### 1. Install dependencies
```bash
pip install numpy matplotlib pandas scipy
```

### 2. Run the model
```bash
python IndAS117_Model.py
```

### 3. Expected runtime
| Section | Time |
|---------|------|
| Sections 1–5 (setup, BEL) | ~30 seconds |
| Section 6 (Monte Carlo, 500 scenarios) | ~2–3 minutes |
| Sections 7–9 (CSM, charts, save) | ~10 seconds |

---

## Model Architecture

### Section 1 — IALM 2012-14 Mortality Table
Exact qx values from the published IAI PDF loaded into Python dictionaries for age-based lookup (ages 2–115). Female rates derived as 78% of male per IRDAI/LIC actuarial practice (LIC Valuation Basis L-42).

### Section 2 — RBI G-Sec Yield Curve
Nine tenor points (1yr to 30yr) interpolated using `scipy.interpolate.CubicSpline` to produce a smooth discount curve. Under Ind AS 117 Para 36, discount rates must reflect **current market rates** at each reporting date.

### Section 3 — Actuarial Functions
Three core functions:
- `get_tpx()` — survival probability vector from entry age
- `epv_death_benefit()` — EPV of claims paid at mid-year of death
- `epv_annuity_due()` — EPV of premium annuity (start of each year alive)

### Section 4 — Portfolio Simulation
1,000 policyholders simulated with:
- Ages 25–55 (uniform random)
- Gender 70% male / 30% female (IRDAI 2024-25 industry mix)
- Sum assured log-normal (mean ₹25L, range ₹5L–₹100L)

### Section 5 — Premiums and BEL
**Net premium** = EPV(claims) / EPV(annuity) — pure equivalence principle  
**Gross premium** = Net / (1 − expense ratio − commission − profit margin)  
**BEL** = EPV(outflows) − EPV(inflows)

Pricing assumptions:
```
Expense ratio   : 15.6%  (IRDAI Annual Report 2024-25)
Commission rate :  5.0%
Profit margin   :  8.0%  → creates positive CSM at inception
```

### Section 6 — Risk Adjustment (Monte Carlo)
Key methodology advantage of Python/R over Excel:

```python
for yr in range(1, term + 1):
    qx    = get_qx(age + yr - 1, gender)
    death = rng.binomial(1, tpx[yr] * qx)   # stochastic death draw
    if death == 1:
        total_pv += sa * disc(yr - 0.5)
        break
```

500 scenarios produce a distribution of total claims PV.  
**Risk Adjustment = 75th percentile − Mean** (VaR approach, Ind AS 117 Para 37)

### Section 7 — CSM at Inception
```
FCF  = BEL + RA + acquisition costs − first-year premium
CSM  = −FCF
```
If CSM < 0 → onerous contract (loss recognised immediately, CSM = 0).

### Section 8 — CSM Roll-Forward (Ind AS 117 Para 44)
```
Opening CSM
+ Interest accretion (locked-in rate = 10-yr G-Sec at inception = 6.53%)
− CSM released (proportional to coverage units delivered)
= Closing CSM
```

CSM reaches ₹0.000 at year 20 — all profit recognised by end of coverage.

---

## Key Results

### Balance Sheet at Inception
```
══════════════════════════════════════════════════════════
  IND AS 117 — INSURANCE CONTRACT LIABILITY AT INCEPTION
══════════════════════════════════════════════════════════
  Best Estimate Liability (BEL)     : ₹   -1.607 crore
  Risk Adjustment (RA) [75th pctile]: ₹    0.997 crore
  Contractual Service Margin (CSM)  : ₹    2.345 crore
──────────────────────────────────────────────────────────
  Total Insurance Contract Liability: ₹    1.735 crore
══════════════════════════════════════════════════════════
```

### CSM Roll-Forward (Selected Years)
| Year | Opening | Interest | Released | Closing |
|------|---------|----------|----------|---------|
| 1    | 2.345   | 0.153    | 0.131    | 2.367   |
| 5    | 2.390   | 0.156    | 0.166    | 2.380   |
| 10   | 2.217   | 0.145    | 0.223    | 2.139   |
| 15   | 1.625   | 0.106    | 0.295    | 1.436   |
| 20   | 0.365   | 0.024    | 0.389    | **0.000** |

All figures in ₹ crore.

---

## Regulatory Context

| Milestone | Date |
|-----------|------|
| Ind AS 117 notified by MCA | August 2024 |
| IRDAI Expert Committee reconstituted | February 2024 |
| First proforma Ind AS 117 submission | December 2025 |
| Second proforma submission | June 2026 |
| Full implementation deadline | **April 2027 (FY27)** |

India joins 140+ countries implementing IFRS 17. The transition affects all 58 Indian insurers, requiring them to move from Indian GAAP to a market-consistent, fulfilment-value basis of reporting.

---

## Limitations and Assumptions

- Female mortality derived as 78% of male (separate female IALM table not publicly available)
- Lapse rates set to zero (no policyholder surrenders modelled)
- No expense inflation — expenses assumed level in real terms
- Portfolio is homogeneous (single product, single term)
- RBI G-Sec rates assumed flat over projection period (locked-in at inception per Ind AS 117)

---

## License

This project is for educational and portfolio purposes. Data sources are all publicly available.  
IALM 2012-14 © Institute of Actuaries of India. RBI data © Reserve Bank of India.

---

*Built as part of an actuarial portfolio project demonstrating Ind AS 117 (IFRS 17) implementation skills for the Indian insurance market.*
