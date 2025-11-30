# Kempower Charging Data — EDA & Clustering (SoH/ETH) Report

## 1. Objective and Data Overview

The goal is to analyze charging-session behavior and infer health-like segments ("EV Transaction Health") based on charging patterns.

### **Data Structure**

- Source file: `dataset_with_battery_size.csv`
- Processing steps:
  - Raw row-level data → aggregated to **transaction-level**.
  - Final dataset contains **90,465** transactions.

### **Key Columns (Transaction-Level)**

**Categorical**
- `country`
- `EVModel`
- `weekday`, `month`, `quarter`, `year`

**Numeric**
- `battery_size`  
- `avgPowerW`, `avgCurrentA`, `avgVoltageV`  
- `soc` (state of charge)  
- `tempC`  
- `sampleTime10sIncrement` (session duration proxy)

---

## 2. Exploratory Data Analysis (EDA)

### **2.1 Summary Statistics**

| Feature | Key Insights |
|--------|--------------|
| **battery_size (kWh)** | Mean ≈ 67.9, Median ≈ 74.0 → fleet dominated by 60–80 kWh packs |
| **avgPowerW (kW)** | Mean ≈ 65.9, heavy right tail up to 225 kW |
| **avgCurrentA (A)** | Broad spread (8–471 A), high-current DC fast charging visible |
| **avgVoltageV (V)** | Mean ≈ 415.5, range 294–795 → mix of 400V & 800V architectures |
| **soc (%)** | Mean ≈ 57%, wide range (4–98%) |
| **tempC (°C)** | Mean ≈ 9.5°C, wide climate range (–35 to 39°C) |
| **sampleTime10sIncrement** | Heavy-tailed: average ~1920 (5.3 hrs), max extremely long |

### **2.2 Missing Values**

All relevant features show **zero missing values** at transaction level — excellent data quality.

### **2.3 Distribution Observations**

- **Power & Current**: Right-skewed, many moderate-power sessions with some very high-power fast charges.
- **Battery Size**: Clustered around common EV pack sizes.
- **Temperature**: Very wide environmental variation.
- **Session Duration**: Heavy-tailed; some extremely long sessions may be anomalies.

### **2.4 Correlation Structure**

- **avgPowerW**, **avgCurrentA**, **avgVoltageV** → strongly correlated (charging physics).
- **battery_size** modestly correlates with voltage and power.
- **session length** tied to SOC and pack sizes (behavior-dependent).

The feature set is appropriate for clustering.

---

## 3. Clustering & Health (ETH/SoH)

### **3.1 Preprocessing**

- One-hot encoding for: `country`, `EVModel`, `weekday`.
- Selected numeric features scaled using **StandardScaler**.
- Clustering performed on:

```
avgPowerW, avgCurrentA, avgVoltageV,
soc, tempC, sampleTime10sIncrement, battery_size
```

### **3.2 K-Means Results**

- **K = 3**, `n_init=10`, `max_iter=300`
- **Silhouette Score = 0.272**

A silhouette of ~0.27 indicates:
- Clusters exist but overlap significantly.
- Structure is moderate, not tight.

### **3.3 Cluster Profiles (Centroids Interpreted)**

| Cluster | Behaviors (Translated from Z-Scores) |
|---------|---------------------------------------|
| **0** | High power (~100 kW), large packs (~77 kWh), moderate SOC (~58%), medium session duration (~4.5 hrs) |
| **1** | High power (~91 kW), large packs (~79 kWh), **lower SOC (~47%)**, shorter sessions (~3.8 hrs) |
| **2** | **Low power (~46 kW)**, small packs (~59 kWh), **higher SOC (~64%)**, long sessions (~6.5 hrs) |

Interpretation:
- **Cluster 0** → Fast charging on big packs, typical mid-SOC sessions.  
- **Cluster 1** → Aggressive fast-charging events starting from low SOC.  
- **Cluster 2** → Slow charges / top-ups / hardware constraints.

### **3.4 SoH (EV Transaction Health) Labeling**

Heuristic rule (in standardized units):

```
Healthy:   soc > 0.8 and avgPowerW > 0.5
Aging:     soc > 0.0 and avgPowerW > 0.0
Degraded:  otherwise
```

Based on centroids:

| Cluster | SoH Label |
|---------|-----------|
| **0** | Aging |
| **1** | Degraded |
| **2** | Degraded |

Findings:
- No cluster satisfies the strict “Healthy” condition.
- Two clusters collapse into “Degraded”, though behaviorally different.

---

## 4. Interpretation & Limitations

### **4.1 What Clusters Actually Represent**
- **Not actual battery physical SoH**  
  They represent *charging behavior patterns*, not internal chemistry degradation.

### **4.2 Limitations**
- Heuristic thresholds operate in **z-score** space (not physical values).
- SOH categories do not map cleanly to clusters → two clusters labeled the same.
- Silhouette score indicates **cluster overlap**.
- Some features (SOC, power) reflect **driver behavior or charger availability**, not battery health.
- Very long session durations likely distort cluster boundaries.

---

## 5. Recommendations & Next Steps

### **1. Redefine SoH Thresholds in Real Units**
Use thresholds like:
- High power > 90 kW  
- SOC bands 20–80%  
- Voltage stability, current taper rate, etc.

### **2. Explore More Cluster Counts**
Test K = 2, 3, 4, 5 with:
- Silhouette scores
- Davies–Bouldin index
- Clear interpretability

### **3. Try Advanced Models**
- **Gaussian Mixture Models (GMM)**  
  For soft boundaries.
- **DBSCAN**  
  To identify truly anomalous or degraded charging events.

### **4. Clean Long/Anomalous Sessions**
Cap or remove extreme durations.

### **5. Fleet/Regional Analysis**
Segment by:
- `EVModel`
- `country`  
To see if specific fleets show more “degraded-like” cluster membership.

---

## 6. Summary


- Aggregates >90k charging sessions into transaction-level analytics.
- Performs thorough EDA showing strong variation in charging behavior.
- Applies K-Means clustering to numeric charging features.
- Finds 3 moderate-quality clusters representing usage patterns:
  - High-power / big-pack sessions  
  - Low-SOC fast charges  
  - Low-power long sessions  
- Uses a heuristic SoH rule to label clusters, though no cluster qualifies as “Healthy”.

With improved thresholds, better clustering model choice, and additional physical features, this approach can evolve into a robust **EV Transaction Health (ETH)** or behavioral SoH estimation tool.
