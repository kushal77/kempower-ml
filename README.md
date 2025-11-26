# Report on EV Charging Dataset Clustering and SoH Labeling

---

## 1. Objective

The notebook performs an exploratory data analysis (EDA) and unsupervised clustering on a 50k-row EV charging dataset (`50Ksample.csv`).  

The main goals are:

- Understand the basic structure and distribution of key operational variables.
- Cluster charging sessions using K-Means on selected numeric features.
- Visualize clusters using PCA (2D projection).
- Assign an approximate **State of Health (SoH)** label to each cluster using simple rules based on cluster centroids.

---

## 2. Data Description

The dataset is loaded and contains **50,000** records with the following key columns:

- **Categorical**
  - `transactionId` – unique session identifier.
  - `country` – e.g., *United Kingdom, Finland, Norway*.
  - `EVModel` – various EV models (Kia, Mercedes, Lexus, etc.).
  - `weekday` – day of the week (string).

- **Temporal / index-like**
  - `year` – mainly 2024–2025.
  - `month` – 1 to 12.
  - `quarter` – 1 to 4.
  - `sampleTime10sIncrement` – an integer-like counter representing time steps (e.g., 10, 470, 960, …).

- **Operational / electrical**
  - `soc` – State of Charge [%], 0–100.
  - `tempC` – temperature [°C], approx −33 to 39.
  - `avgPowerW` – average power in watts (mean ≈ 58,372 W).
  - `avgCurrentA` – average current (mean ≈ 144 A).
  - `avgVoltageV` – average voltage (mean ≈ 406 V).

### 2.1 Summary Statistics (Selected)

From `df.describe()`:

- `soc`  
  - Mean ≈ 59.6, std ≈ 22.3, range 0–100.

- `tempC`  
  - Mean ≈ 8.2°C, std ≈ 9.0°C, min −33°C, max 39°C.

- `sampleTime10sIncrement`  
  - Mean ≈ 1173.7, std ≈ 928.3, captures session length in 10s units.

- `avgPowerW`  
  - Mean ≈ 58.4 kW, std ≈ 33.8 kW, indicating a wide spread of charging power.

- `avgCurrentA`, `avgVoltageV`  
  - Both show variation consistent with different EV and charger configurations.

### 2.2 Missing Values

- `df.isnull().sum()` indicates **zero missing values** across all columns.  
- No imputation was required in the current workflow.

---

## 3. Exploratory Data Analysis (EDA)

The notebook includes basic univariate and multivariate EDA.

### 3.1 Univariate Distributions

Using `sns.histplot` with KDE for:

- `avgPowerW`
- `avgCurrentA`
- `avgVoltageV`

From the plotted distributions:

- `avgPowerW` shows a broad distribution around typical fast-charging power levels with a long tail towards higher power.
- `avgCurrentA` is widely spread, reflecting different charging speeds and station capabilities.
- `avgVoltageV` is more concentrated, consistent with typical EV pack voltages, but still with some spread due to varying chemistries and architectures.

(These can be extended later to include `soc`, `tempC`, and `sampleTime10sIncrement`.)

### 3.2 Correlation Analysis

A correlation matrix is computed for:

```python
['avgPowerW', 'avgCurrentA', 'avgVoltageV', 'soc', 'tempC', 'sampleTime10sIncrement']
