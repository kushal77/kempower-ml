# EV Charging Dataset -- EDA, Clustering, and SoH Labeling Review

## 1. Objective

This notebook performs exploratory data analysis (EDA) and unsupervised
clustering on a 50k-row EV charging dataset (`50Ksample.csv`). The main
goals are:

-   Examine basic structure and distribution of key operational
    variables.
-   Apply **K-Means clustering** on selected numeric features.
-   Visualize clusters using **PCA** (2D projection).
-   Assign approximate **State of Health (SoH)** labels to clusters.

## 2. Data Description

### Categorical Features

-   `transactionId`
-   `country`
-   `EVModel`
-   `weekday`

### Temporal Features

-   `year`, `month`, `quarter`
-   `sampleTime10sIncrement`

### Operational / Electrical

-   `soc`, `tempC`, `avgPowerW`, `avgCurrentA`, `avgVoltageV`

Dataset had zero missing values and expected statistical ranges.

## 3. EDA Summary

-   Histograms for power, current, voltage.
-   Correlation matrix of major numeric fields.
-   Patterns consistent with fast-charging behaviors.

## 4. Preprocessing

-   Dropped missing values.
-   One-hot encoding of categorical variables.
-   Selected continuous features for clustering.
-   StandardScaler applied.

## 5. Clustering

### K-Means (k=3)

-   Applied on standardized numeric variables.
-   PCA used for visualization.

### Silhouette Score

    0.229

→ Weak cluster separation.

## 6. SoH Labeling Issue

The notebook compares **standardized centroids** with **raw-unit
thresholds**, causing all clusters to be labeled **Degraded**.

Fix required: evaluate centroids in raw scale or adjust thresholds to
z-score units.

## 7. Recommended Improvements

-   Use raw centroids for SoH rules.
-   Optimize k using elbow or silhouette methods.
-   Consider clustering with categorical variables.
-   Better modularization of code.
-   Improve EDA with more segmentation.

## 8. Conclusion

The notebook is well-structured but the SoH logic is invalid due to
scale mismatch. Clustering structure is weak and requires optimization
for meaningful interpretation.
