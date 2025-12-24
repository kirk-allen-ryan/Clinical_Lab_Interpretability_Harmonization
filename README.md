## 📊 Site Characteristics & Population Acuity

### 📉 Table: Distribution of Published Lower Limits (LL) by Site
*Dataset: 11,382 Serum Calcium results over 12 months.*

| Site Location | Total N | Dominant LL (mg/dL) | Variance in LL (Range) | % Fragmentation |
| :--- | :---: | :---: | :---: | :---: |
| **Boston** | 490 | 8.5 | Fixed (8.5) | 0% |
| **Chicago** | 638 | 9.0 | 8.0 — 9.3 | 14.7% |
| **Cincinnati** | 527 | 8.4 | 8.0 — 8.5 | 65.2% |
| **Galveston** | 6,100 | 9.0 | 8.0 — 9.3 | **High (8 Tiers)** |
| **Greenville** | 64 | 9.0 | 8.5 — 9.1 | 28.1% |
| **Northern Cal** | 2,762 | 9.0 | 8.1 — 9.3 | 30.1% |
| **Philadelphia** | 636 | 9.1 | 8.0 — 9.3 | 22.8% |
| **Portland** | 128 | 8.5 | Fixed (8.5) | 0% |
| **Grand Total** | **11,382** | **9.0** | **8.0 — 9.3** | **Global Split** |

### ⚠️ Case Study: Intra-Patient Interpretive Discordance (Galveston)
*Analysis of individual patient encounters where Calcium results were evaluated against multiple, conflicting Lower Limits (LL) during a single stay.*

| Patient ID | Total Results | LL Tier A (n) | LL Tier B (n) | Delta in LL | Impact |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **868** | 82 | **8.1** (35) | **8.7** (47) | 0.6 mg/dL | Flag Jitter |
| **976** | 28 | **9.0** (24) | **9.3** (4) | 0.3 mg/dL | Inconsistent Flagging |
| **326** | 82 | **8.0** (81) | **9.0** (1) | 1.0 mg/dL | Major Threshold Shift |
| **752** | 608 | **9.0** (61) | **9.3** (547) | 0.3 mg/dL | High-Volume Noise |

**Key Finding:** Patients are subjected to "interpretive drift" where the definition of "Normal" shifts mid-treatment, leading to non-actionable alerts and diagnostic confusion.

Sample,Mean (mg/dL),Analytical CV (%),Allowable Margin (mg/dL),Allowed % Error to Pass
1,9.06,2.10%,±1.05,±11.6%
2,12.30,2.11%,±1.00,±8.1%
3,7.72,2.33%,±1.05,±13.6%
4,9.71,2.16%,±1.05,±10.8%
5,10.39,2.02%,±1.05,±10.1%

Key Takeaways for your Grant/StudyThe "Tightness" Paradox: Your lab’s Analytical CV is very consistent (roughly $2.1\%$). This means your instrument is highly precise. However, the regulatory "Passing Window" is much wider (up to $13.6\%$ at low levels).Why this supports your "Noise" theory: Because regulators allow a $\pm 1.0\text{ mg/dL}$ error, a lab can "Pass" proficiency testing even if they are consistently $0.8\text{ mg/dL}$ higher than the peer group. If that lab then sets their reference range based on that biased mean, they create the exact "Interpretive Fragmentation" you found in your data.The $1.0\text{ mg/dL}$ Delta: This $\pm 1.0$ limit is massive for Calcium. In your Galveston/GC-1 data, you saw lower limits moving from $8.0$ to $9.3$. That $1.3\text{ mg/dL}$ spread is almost exactly the "allowable error" defined by CLIA.Conclusion: You can argue in your GitHub README that "Regulatory Compliance ($\pm 1.0\text{ mg/dL}$) is too permissive for an analyte with a $2.0\text{ mg/dL}$ total physiological range, effectively codifying interpretive discordance into law."


import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# 1. Load the "Git-Safe" Scrubbed Data
# Ensure your file is named 'ca_study_data.csv' in the same folder
df = pd.read_csv('ca_study_data.csv')

# 2. Define the "Gold Standard" Harmonized Range
# We use 8.5 - 10.5 mg/dL as the clinical consensus benchmark
GOLD_LOW = 8.5
GOLD_HIGH = 10.5

# 3. Calculate "Discordance"
# This identifies results that were flagged differently than the gold standard
df['Flag_Agreement'] = (df['Result_Val'] < df['Lower_Limit']) == (df['Result_Val'] < GOLD_LOW)

# 4. Filter for the "Interpretive Flip-Flop"
# Finding patients who had >1 unique lower limit during their stay
patient_stats = df.groupby('Patient_ID_Alpha').agg({
    'Lower_Limit': 'nunique',
    'Result_Val': 'count',
    'Region_ID': 'first'
}).rename(columns={'Lower_Limit': 'Unique_LL_Count'})

flip_flop_patients = patient_stats[patient_stats['Unique_LL_Count'] > 1]

print(f"Project Ready: {len(df)} results loaded.")
print(f"Alert Fatigue Signal: {len(flip_flop_patients)} patients experienced shifting limits.")
