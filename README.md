# Doubling to 20B VND — E-commerce Growth Plan  
**Plan B: 3 Super-Events + ROI-Gated Traffic Allocation (STRICT v3b)**

> **Objective:** Seller A reached ~10B VND **gross** in July. **August KPI = 20B VND**.  
> **Deliverable:** A quantified, executable plan to achieve 20B with clear pacing, budget/ROI guardrails, and risk controls.

> **Assets layout (adjust links if needed):**
> ```
> assets/
>   images/   # charts & figures (PNG)
>   tables/   # CSVs for tables
>   docs/     # narrative / flow analysis
> ```

---

## Table of Contents
1. [Executive Summary](#executive-summary)  
2. [Data & Quality Assurance](#data--quality-assurance)  
3. [Core Equations & Definitions (Correct)](#core-equations--definitions-correct)  
4. [EDA: What the Data Says](#eda-what-the-data-says)  
5. [Baseline & Super-Event Modeling](#baseline--super-event-modeling)  
6. [Channel Diagnosis (SCALE / FIX / PAUSE)](#channel-diagnosis-scale--fix--pause)  
7. [Allocation Strategy — STRICT v3b (ROI-Gated)](#allocation-strategy--strict-v3b-roi-gated)  
8. [Revenue Bridge to 20B](#revenue-bridge-to-20b)  
9. [Weekly→Daily KPI & Pacing](#weeklydaily-kpi--pacing)  
10. [Risk & Sensitivity](#risk--sensitivity)  
11. [Experiments to Unlock FIX Uplift](#experiments-to-unlock-fix-uplift)  
12. [Operating Rhythm & Governance](#operating-rhythm--governance)  
13. [How to Reproduce](#how-to-reproduce)  
14. [Suggested Repository Structure](#suggested-repository-structure)  
15. [License & Notes](#license--notes)  
16. [Appendix — Notation](#appendix--notation)

---

## Executive Summary

We decompose revenue and quantify each lever:

- **Clean baseline (30d)** on **Completed** revenue, **excluding** the super-event outlier → ~**4.38B** VND.
- **Per super-event uplift** ~**2.79B** VND → **3 events ≈ +8.36B**.
- **Residual GAP** before traffic: ~**7.26B** → closed via **ROI-gated traffic allocation (STRICT v3b)**.
- **Resulting plan** ≈ **20B** (GAP = 0) with pacing & risk controls.

**Bridge (slide-ready):**  
![Revenue Bridge](assets/images/revenue_bridge_v3b.png)

---

## Data & Quality Assurance

**Sources**
- **OMS Orders**: order lines (status, payment method, shipping, original/selling price).  
- **GA Traffic**: Source/Medium, Sessions, Transactions, Conversion Rate, Revenue.

**Critical QA**
- Removed **GA `GRAND TOTAL`** (last row).  
- OMS–GA reconciliation (illustrative):
  - GA **Gross Revenue (clean)** ≈ **10,006,473,136** VND  
  - OMS **Completed product revenue** ≈ **8,072,083,720** VND  
  - OMS **Cancelled product revenue** ≈ **1,993,771,450** VND  
  - **Shipping fee** (all / completed) ≈ **690,804,572 / 508,978,481** VND

Artifacts:
- Channel diagnosis: [`assets/tables/channel_diagnosis_actions.csv`](assets/tables/channel_diagnosis_actions.csv)

---

## Core Equations & Definitions (Correct)

> **All ratios are computed on the same scope/window (e.g., per channel or day).**  
> **Handle zeros safely** (e.g., define AOV only when Transactions > 0).

### 3.1 Revenue Identity
Daily (or per segment) identity and 30-day sum:
$$
R \;=\; \sum_{d=1}^{D} S_d \cdot \mathrm{CVR}_d \cdot \mathrm{AOV}_d
$$
Where:
- \(R\) = revenue over the period,
- \(S_d\) = sessions on day \(d\),
- \(\mathrm{CVR}_d = \frac{T_d}{S_d}\) if \(S_d>0\) (else 0),
- \(\mathrm{AOV}_d = \frac{R_d}{T_d}\) if \(T_d>0\) (else undefined; skip or impute).

### 3.2 Conversion, AOV, RPS
Conversion and order value:
$$
\mathrm{CVR} \;=\; \frac{\text{Transactions}}{\text{Sessions}}, \qquad
\mathrm{AOV} \;=\; \frac{\text{Revenue}}{\text{Transactions}} \;(\text{if Transactions}>0)
$$

Revenue per session (proxy):
$$
\mathrm{RPS} \;=\; \frac{\text{Revenue}}{\text{Sessions}} \;=\; \mathrm{CVR}\times \mathrm{AOV}
$$

### 3.3 Event Uplift (per day and scenario)
Per super-event day uplift (baseline-adjusted):
$$
\mathrm{Uplift}_{\text{event\,day}} \;=\; R_{\text{actual}} \;-\; \widehat{R}_{\text{baseline}}
$$
For \(n\) super-events:
$$
R_{\text{events}} \;=\; \sum_{i=1}^{n} \mathrm{Uplift}_i
$$

### 3.4 Traffic-Added Revenue (per channel)
For channel \(c\) with incremental sessions \(\Delta S_c\):
$$
\Delta R_c \;=\; \Delta S_c \cdot \mathrm{RPS}_c \;=\; \Delta S_c \cdot \mathrm{CVR}_c \cdot \mathrm{AOV}_c
$$

### 3.5 ROI Gate (per session)
We gate PAID scaling by **ROI per session**:
$$
\mathrm{ROI}_{ps} \;=\; \frac{\mathrm{RPS}}{\mathrm{CPS}} \;\ge\; \theta
$$
- \(\mathrm{CPS}=\frac{\text{Spend}}{\text{Sessions}}\) (**cost per session**).  
- In absence of spend/clicks in GA, we **approximate** \(\mathrm{CPS}\approx \mathrm{CPC}\cdot\frac{\text{Clicks}}{\text{Sessions}}\).  
  If we assume \(\text{Clicks}\approx\text{Sessions}\), then \(\mathrm{CPS}\approx\mathrm{CPC}\).  
- We use \(\theta=1.10\) (≥10% unit margin buffer).

> **Assumption disclosure:** The CPC→CPS approximation is a practical proxy due to data limits; replace with true **Spend/Sessions** when available.

### 3.6 Caps (capacity constraints)
Let \(S_c\) be current sessions of channel \(c\).

- **OWNED** (tight cap):
$$
0 \;\le\; \Delta S_c \;\le\; \min\!\Big(S_c\cdot(\mathrm{CAP}_{OWNED}-1),\; \mathrm{ABS\_OWNED\_ADD}\Big)
$$

- **PAID** (looser cap on the *additional* part):
$$
0 \;\le\; \Delta S_c \;\le\; S_c\cdot(\mathrm{CAP}_{PAID}-1)
$$

### 3.7 CVR Significance (two-proportion z-test)
With successes \(x_1, x_2\) and trials \(n_1, n_2\), pooled rate \(p=\frac{x_1+x_2}{n_1+n_2}\):
$$
z \;=\; \frac{\hat p_1 - \hat p_2}{\sqrt{p(1-p)\left(\frac{1}{n_1}+\frac{1}{n_2}\right)}}, 
\quad \hat p_i = \frac{x_i}{n_i}
$$

### 3.8 A/B Sample Size (CVR uplift, equal allocation)
For target absolute lift \(\Delta = p_1 - p_2\), with \(z_{1-\alpha/2}\) and \(z_{1-\beta}\):
$$
n_{\text{per arm}} \;\approx\; 
\frac{2\big(z_{1-\alpha/2}+z_{1-\beta}\big)^2\;\bar p(1-\bar p)}{\Delta^2},
\qquad \bar p=\frac{p_1+p_2}{2}
$$

---

## EDA: What the Data Says

- **Daily (Completed)** — stable base with a single sharp **super-event** spike.  
  ![Daily Completed Revenue](assets/images/eda_daily_revenue.png)
- **By Day-of-Week** — no material systematic DOW effect (Kruskal–Wallis non-significant).  
  ![Revenue by DOW](assets/images/eda_dow_revenue.png)
- **Super-event detection** — flagged and **excluded** from baseline training.  
  ![Super-Event Flag](assets/images/event_flagged_daily_revenue.png)

**Takeaway:** Demand is primarily trend-driven; **super-events** produce discontinuous spikes. Use events to lift volume, then close the residual GAP via traffic allocation.

---

## Baseline & Super-Event Modeling

- **Baseline**: Prophet on **Completed** revenue, **excluding** the super-event day → **~4.38B** VND (30d).  
  ![Prophet Forecast](assets/images/prophet_forecast_plot.png)
- **Per super-event uplift** ≈ **2.79B** VND (baseline-adjusted).  
  Scenarioing (0→6 events) supports **3 events** as a feasible/risk-balanced Plan B.

---

## Channel Diagnosis (SCALE / FIX / PAUSE)

- **RPS** \(=\mathrm{CVR}\times\mathrm{AOV}\) and **CVR z-test vs rest** drive labels:
  - **SCALE**: strong CVR (significant & ≥ median), strong RPS, low bounce  
  - **FIX**: material volume but sub-par RPS/CVR → fix LP/creative/offer/checkout  
  - **PAUSE**: long-tail or under-performing

See: [`assets/tables/channel_diagnosis_actions.csv`](assets/tables/channel_diagnosis_actions.csv)

---

## Allocation Strategy — STRICT v3b (ROI-Gated)

**Priority:** `SCALE (1) → FIX (2) → PAID ROI-pass (2.5)` where \(\mathrm{ROI}_{ps}\ge\theta\).  
**Caps:** OWNED tight, PAID ≤ +3× *additional* sessions.  
**FIX uplift (pre-allocation):** apply **CVR +10%** and **AOV +3%** to FIX channels (to be validated by A/B).

Outputs:
- Master allocation — [`assets/tables/strict_v3b_master_allocation.csv`](assets/tables/strict_v3b_master_allocation.csv)  
- Summary by type — [`assets/tables/strict_v3b_alloc_summary.csv`](assets/tables/strict_v3b_alloc_summary.csv)  
- Budget estimate (CPC heuristics: Search≈1400, Social≈900, Video≈800 VND) — [`assets/tables/budget_v3b.csv`](assets/tables/budget_v3b.csv)

**Why it works:** The **ROI gate** ensures incremental sessions have positive unit economics while still **unlocking** high-ROI PAID (e.g., Google/YouTube) that might be undervalued by raw labels.

---

## Revenue Bridge to 20B

30-day bridge:
- **Baseline**: ~**4.38B**  
- **+ 3 Super-events**: ~**+8.36B**  
- **+ Traffic (STRICT v3b)**: ~**+7.26B**  
= **~20B** (**GAP = 0**)

![Bridge](assets/images/revenue_bridge_v3b.png)  
Data: [`assets/tables/revenue_bridge_v3b.csv`](assets/tables/revenue_bridge_v3b.csv)

---

## Weekly→Daily KPI & Pacing

- **Weekly split** aligned to event cadence/paydays: **W1 30% – W2 0% – W3 30% – W4 40%**.  
- **Daily distribution** by DOW weights learned from July (excluding super-event).

Artifacts:
- Weekly KPI — [`assets/tables/weekly_kpi_v3b.csv`](assets/tables/weekly_kpi_v3b.csv)  
- Day-level KPI — [`assets/tables/day_level_kpi_v3b.csv`](assets/tables/day_level_kpi_v3b.csv)  
- Planned daily added revenue — ![Daily Planned Revenue](assets/images/daily_planned_revenue_v3b.png)

---

## Risk & Sensitivity

**Example downside:** PAID \(\mathrm{CVR}\) drops by \(\delta=10\%\).  
Recompute per channel:
$$
\mathrm{RPS}'_c \;=\; (1-\delta)\cdot \mathrm{CVR}_c \cdot \mathrm{AOV}_c
$$
If a GAP reappears, **reallocate** to highest \(\mathrm{ROI}_{ps}\) and trigger **CRM pushes** + **AOV/CVR uplifts** (bundles/threshold, LP speed/trust).

(If exported): [`assets/tables/sensitivity_v3b.csv`](assets/tables/sensitivity_v3b.csv)

---

## Experiments to Unlock FIX Uplift

- **CVR A/B (two-proportion test)** — required sample per arm:
$$
n_{\text{per arm}} \;\approx\; 
\frac{2\big(z_{1-\alpha/2}+z_{1-\beta}\big)^2\;\bar p(1-\bar p)}{\Delta^2},
\;\; \bar p=\frac{p_1+p_2}{2}
$$
- **AOV uplift** — non-normal: use **Mann–Whitney** or **bootstrap**; report **median** & **upper quantiles**.

---

## Operating Rhythm & Governance

- **Daily (AM)**: Nowcast vs daily KPI (Sessions, CVR, AOV, RPS, Revenue). If off by >±5–10%, **reallocate** within ROI gates; verify stock & LP/checkout SLA.  
- **Weekly (1-pager)**: Bridge position, top channels, test status, risks & next-week actions.  
- **Event days**: inventory assurance, bid/creative ramps, LP/checkout readiness; post-event remarketing.

Artifacts (optional):
- Flow analysis — [`assets/docs/flow_analysis.md`](assets/docs/flow_analysis.md)  
- Key metrics roll-up — [`assets/tables/key_metrics_summary.csv`](assets/tables/key_metrics_summary.csv)  
- Top channels — [`assets/tables/top_channels_by_rev_add.csv`](assets/tables/top_channels_by_rev_add.csv)


