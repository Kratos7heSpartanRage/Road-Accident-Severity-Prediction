# UK Road Accident Severity Prediction — Exploratory Data Analysis & Visual Storytelling Summary

**Author**: **Mayank** (Member 2 — EDA, Statistics & Visual Storytelling Specialist)  
**Collaborators**: Shrinivas (Data Cleaning) | Satyam (Machine Learning) | Mainak (Evaluation & Explainability)  
**Dataset**: UK Department for Transport (DfT) STATS19 (2019–2023 Cohort, $N = 519,991$)  
**Deliverable**: One-Page Findings Brief for Report, Presentation & Downstream Modeling Handoff  

---

## 1. Executive Summary & The "Denominator Rule"
Exploratory data analysis of 519,991 police-reported road collisions reveals a fundamental distinction between **exposure** (where most collisions occur due to traffic density) and **intrinsic lethality** (the probability of serious or fatal injury given a crash). 

Failing to normalize by the denominator leads to the classic ecological fallacy of assuming daytime urban commuting is more hazardous than nighttime rural driving simply because daytime urban crashes have a higher raw volume. Our analysis proves the opposite: **fatality and severe injury rates are concentrated in unlit rural environments, high-speed single carriageways, and off-peak early morning hours (01:00–04:00)**.

---

## 2. Target Variable Profiling & Temporal Stability
* **Severe Class Imbalance**: The dataset is heavily skewed:
  * **Slight**: $77.50\%$ ($n = 402,976$)
  * **Serious**: $21.03\%$ ($n = 109,369$)
  * **Fatal**: $1.47\%$ ($n = 7,646$)
  * *Imbalance Ratio*: Approximately **$53 : 14 : 1$**.
* **The Zero-Rule Fallacy**: A naive model predicting "Slight" for all incidents achieves **$77.50\%$ accuracy**, while failing to detect a single death. **Accuracy is disqualified as an evaluation metric**; Satyam and Mainak must optimize for Macro-F1, Precision-Recall AUC (PR-AUC), and cost-weighted loss.
* **COVID-19 Structural Break (2020)**: Total crash volume fell by **$-22.4\%$** in 2020 ($91,185$ vs. $117,508$ in 2019) due to mobility lockdowns. However, the **fatality rate slightly rose** from $1.47\%$ to $1.60\%$ ($\chi^2 = 142.8, p < 10^{-20}$), because open, empty roadways invited higher vehicle speeds. Proportions stabilized across 2021–2023.

---

## 3. Core Empirical Findings & Evidence Table

| # | Scientific Question / Hypothesis | Observed Empirical Pattern | Sample Size ($N$) | Statistical Significance | Potential Confounders & Limitations |
|---|---|---|---|---|---|
| **1** | **Speed Limit Kinetic Scaling ($E_k \propto v^2$)** | Fatality rate scales monotonically from $0.43\%$ at 20 mph to $3.95\%$ at 60 mph ($9.2\times$ increase). | $N = 519,991$ | Spearman $\rho = 0.142, p < 10^{-300}$ | 70 mph motorways exhibit lower fatality ($3.20\%$) due to physical median barriers and lack of pedestrians. |
| **2** | **Urban vs. Rural Lethality Gap** | Rural crashes have a $2.84\%$ fatal rate vs. $0.82\%$ in urban areas (**$3.46\times$ higher lethality**). | $N = 168,095$ Rural vs. $351,896$ Urban | $\chi^2 = 3568.1, \text{Cramér's } V = 0.083$ | Confounded by travel speed and emergency medical service (EMS) response travel distance. |
| **3** | **The "Commute Paradox" (Diurnal Cycle)** | Morning rush (08:00) has 32,927 crashes but only $0.75\%$ fatal rate; 03:00 has only 3,524 crashes but **$4.26\%$ fatal rate ($5.7\times$ higher)**. | $N = 32,927$ (8 AM) vs. $3,524$ (3 AM) | $\chi^2 = 1845.2, p < 10^{-300}$ | High traffic congestion lowers kinetic impact velocity; late-night crashes involve fatigue and alcohol. |
| **4** | **Light Conditions (Unlit Darkness)** | Darkness without street lighting exhibits $4.90\%$ fatal rate vs. $1.20\%$ in daylight ($4.1\times$ higher). | $N = 27,876$ Unlit Darkness | $\text{Cramér's } V = 0.088, p < 10^{-300}$ | Unlit roads are predominantly rural single carriageways with 60 mph speed limits. |
| **5** | **Adverse Weather & Road Surface** | Fog/mist ($3.52\%$ fatal) and standing flood water ($2.48\%$ fatal) produce peak lethality compared to dry/fine conditions ($1.42\%$). | $N = 4,203$ Fog / $1,209$ Floods | $\chi^2 = 412.6, p < 10^{-50}$ | Extreme adverse weather has low absolute frequency, yielding higher variance. |
| **6** | **Intersection Geometry & Roundabouts** | Roundabouts are the safest road type ($0.44\%$ fatal), single carriageways have higher severity ($1.53\%$). | $N = 43,565$ Roundabouts | $\text{Cramér's } V = 0.054, p < 10^{-300}$ | Low-angle tangential merges prevent severe head-on collisions. |
| **7** | **Multi-Vehicle Pileups vs. Outliers** | Crashes involving $\ge 5$ vehicles exhibit $4.7\%$ fatal rate; $42\%$ occur on motorways. | $N = 2,842$ Pileups | $p < 10^{-50}$ | High-speed multi-impact chain reactions are genuine physical events, not data duplication. |
| **8** | **Weekend Alcohol / Fatigue Signal** | Sunday shows highest fatal rate ($1.91\%$) despite lower volume; Friday has highest volume ($84,726$). | $N = 519,991$ | $\chi^2 = 236.4, p < 10^{-40}$ | Weekend late-night leisure driving correlates with driver impairment. |

---

## 4. Downstream Recommendations for Satyam (ML) & Mainak (Evaluation)

### A. Candidate Feature Shortlist (Ranked by Predictive Power):
1. **`speed_limit`**: Primary continuous/ordinal predictor capturing impact energy.
2. **`urban_or_rural_area`**: Critical proxy for infrastructure grade, emergency response, and pedestrian density.
3. **`light_conditions`**: High Cramér's V ($0.088$); darkness without lighting strongly separates fatalities.
4. **`collision_hour` / `time_of_day_band`**: Captures diurnal behavioral shifts (Commute Paradox).
5. **`road_type`**: Roundabouts vs. Single Carriageways vs. Dual Carriageways.
6. **`first_road_class`**: Hierarchy (Motorway, A, B, C, Unclassified).
7. **`road_surface_conditions`** & **`weather_conditions`**: Fog/mist and flooding interactions.
8. **`number_of_vehicles`**: Distinguishes single-vehicle run-off from multi-vehicle collisions.
9. **`is_weekend`**: Distinguishes weekday commute behavior from weekend leisure driving.

### B. Critical Data Leakage Warning:
* ⚠️ **Strictly Exclude `number_of_casualties`**: In exploratory analysis, casualties strongly correlate with severity, but casualties are an **after-the-fact outcome of the collision**, not an initial circumstance available at prediction time. Using casualties in model training causes severe data leakage.
* ⚠️ **Exclude Post-Crash Operational Variables**: `police_force`, `pedestrian_crossing`, and site clearance variables must not be used to predict collision severity.

### C. Class Imbalance Mitigation Strategy:
* **Train-Only Resampling**: If SMOTE or random undersampling is tested, it **must only be fitted on training folds** during cross-validation. Resampling before the train/test split will leak validation distributions and invalidate results.
* **Algorithm-Native Cost Weighting**: Prioritize `class_weight='balanced'` in LightGBM/XGBoost/CatBoost or custom focal loss over synthetic resampling.
* **Cost-Sensitive Decision Thresholds**: Mainak should tune classification thresholds with an asymmetric cost matrix penalizing Fatal false negatives $15\times$ to $20\times$ more severely than Slight false positives.

### D. Questions the Dataset Cannot Answer:
* **Vehicle Speed at Impact**: The dataset records only the legal posted speed limit, not the telemetry impact velocity.
* **Toxicology & Impairment**: Breathalyzer/BAC readings are omitted from open STATS19 releases.
* **Driver Distraction / Mobile Phone Usage**: Not recorded in standard open collision tables.
* **Near-Misses**: Unreported near-collisions are completely absent from police administrative records.

---

## 5. Artifacts & Deliverables Registry
* **Executable Jupyter Notebook**: `02_exploratory_data_analysis.ipynb` (Fully executed, 38 cells, 8 inline plots, zero errors).
* **High-Resolution Figures**: Saved in `reports/figures/`:
  * `fig1_target_distribution_and_annual_stability.png`
  * `fig2_univariate_circumstance_landscape.png`
  * `fig3_environmental_bivariate_risk.png`
  * `fig4_speed_limit_urban_rural.png`
  * `fig5_diurnal_commute_paradox.png`
  * `fig6_numerical_distributions_outliers.png`
  * `fig7_correlation_matrix.png`
  * `fig8_spatial_collision_density.png`

