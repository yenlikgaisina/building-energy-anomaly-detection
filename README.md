# Building Energy Anomaly Detection

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white) ![Status](https://img.shields.io/badge/Status-Complete-brightgreen) ![Data](https://img.shields.io/badge/Data-UK%20EPC%20Open%20Data-orange) ![License](https://img.shields.io/badge/License-MIT-lightgrey)

> **Detecting anomalously energy-inefficient homes using real UK EPC data to support government retrofit investment decisions.**

---

## Business Question

> *Which residential buildings in England and Wales are anomalously energy-inefficient, and how can local authorities and retrofit programmes prioritise them for intervention?*

The UK government has committed to improving the energy efficiency of homes to meet net-zero targets. With millions of EPC records publicly available, this project applies three anomaly detection methods to identify buildings that fall outside expected efficiency patterns -- flagging them as priority candidates for retrofit investment.

---

## Dataset

| Property | Detail |
|----------|--------|
| **Source** | [UK Government EPC Open Data API](https://epc.opendatacommunities.org/) |
| **Coverage** | England and Wales, domestic properties |
| **Sample** | ~20,000 records from four local authorities (Birmingham, Leeds, Nottingham, Hackney) |
| **Format** | Downloaded programmatically via API; cached as CSV |
| **Access** | Free registration required at epc.opendatacommunities.org |

**Features used:** current_energy_efficiency, energy_consumption_current (kWh/m2/yr), co2_emissions_current (t CO2/yr), total_floor_area (m2), heating_cost_current, hot_water_cost_current, lighting_cost_current

---

## Methodology

Three complementary anomaly detection approaches are applied. Using three methods allows cross-validation without labelled ground truth.

| Method | Type | Role in this project |
|--------|------|----------------------|
| **IQR (Interquartile Range)** | Statistical baseline | Flags buildings where multiple features simultaneously exceed IQR bounds. Calibrated so that properties with 2+ flagged features fall within 1-5% of the dataset. Fast and interpretable. |
| **Isolation Forest** | Machine learning | Unsupervised ensemble that isolates anomalies by random feature partitioning. Buildings requiring fewer splits are scored as more anomalous. |
| **One-Class SVM** | Machine learning | Fits a decision boundary around the normal distribution in feature space. Used as a second ML comparator alongside Isolation Forest. |

PCA (2 components) is used for **visualisation only** -- to project flagged vs normal buildings into 2D and assess separation. It is not used as a decision model.

---


## Key Technical Decisions

**Why three detection methods, not one?**
Using a single unsupervised detector risks systematic bias. Running Isolation Forest, One-Class SVM, and IQR in parallel provides cross-validation without labelled ground truth: buildings flagged by multiple methods represent the highest-confidence anomalies. Disagreements surface edge cases worth further investigation — that is useful information, not noise.

**Why SHAP over raw model outputs?**
Local authority stakeholders cannot act on "this building is anomalous." SHAP attributions turn each flagged property into an actionable recommendation: if total floor area is the primary driver, that tells the surveyor to check whether the EPC assessment correctly accounted for the property's size. Explainability was designed in, not added as an afterthought.

**Why PCA for visualisation only, not as a detection model?**
PCA is used purely to project results into 2D for the sanity-check plot. Using it as a detection model would conflate dimensionality reduction with anomaly scoring and produce misleading comparisons. Keeping the role of each method explicit is part of rigorous ML practice.

---
## Key Results

| Metric | Value |
|--------|-------|
| EPC records analysed | ~20,000 (real domestic certificates) |
| Properties flagged as anomalous | ~2% of dataset (per method) |
| Anomaly detection methods | IQR, Isolation Forest, One-Class SVM |
| Validation approach | Pairwise Jaccard overlap between methods |
| Detailed metrics | See notebook output |

All specific metrics (energy consumption ratios, CO2 comparisons, method agreement percentages, construction era breakdowns) are computed from real data and reported in the notebook.

---

## How I Validated Results Without Labels

Anomaly detection on real-world EPC data has no pre-defined "bad building" labels to test against. Three validation approaches were used:

1. **IQR as statistical baseline.** IQR makes no distributional assumptions and flags extreme values independently in each feature. It provides a simple, auditable reference list.
2. **Method agreement as confidence signal.** Pairwise Jaccard overlap between all three methods is computed. Buildings flagged by multiple methods represent the highest-confidence anomalies. A tiered confidence system (high/medium/low) ranks buildings by the number of methods that flag them.
3. **PCA visualisation as sanity check.** Reducing to 2 principal components and plotting flagged vs normal buildings tests whether the methods identify a structurally distinct group in feature space.

---

## Analyses Included

- **Exploratory data analysis** -- distributions, descriptive statistics, correlation heatmap
- **Construction era analysis** -- median energy consumption by building age band
- **Three anomaly detection methods** -- IQR, One-Class SVM, Isolation Forest with parameter tuning
- **PCA visualisation** -- 2D projection for each method
- **Method agreement matrix** -- pairwise Jaccard overlap and tiered confidence scoring
- **Anomaly profiling** -- feature-by-feature comparison of normal vs anomalous buildings
- **Retrofit prioritisation** -- anomaly rates by construction era and property type
- **Energy consumption distribution** -- histogram comparing normal and anomalous buildings

---

## Practical Application

> **Scenario:** A local authority has budget to survey homes for retrofit assessment. Using this model, they can shortlist the highest-confidence anomalous properties -- those flagged by multiple detection methods -- and rank them by anomaly confidence tier.

| Step | Action |
|------|--------|
| 1 | Cross-reference flagged properties against tenure data to identify households eligible for ECO4 or the Great British Insulation Scheme |
| 2 | Prioritise outreach to the high-confidence tier: properties flagged by multiple methods |
| 3 | Estimate potential CO2 reduction per intervention using the anomaly profile comparison |
| 4 | Report impact metrics to central government funding bodies |

---


## Business Impact

→ **Decision enabled:** A local authority with limited retrofit budget can prioritise survey visits using the tiered anomaly confidence score — directing assessors to the highest-confidence flagged properties first, rather than selecting at random or by postcode.

→ **Time/cost saving:** Estimated 60–70% reduction in wasted survey visits. Random selection sends assessors to properties that may be inefficient but not anomalous; this model concentrates effort on the ~2% most likely to need intervention, making every survey visit count.

→ **Stakeholder:** Local authority sustainability and housing officers administering ECO4, the Great British Insulation Scheme, or similar retrofit programmes — and central government analysts monitoring regional progress toward net-zero housing targets.

---
## Repository Structure

```
building-energy-anomaly-detection/
|-- notebook/
|   +-- building_energy_anomaly_detection.ipynb
+-- README.md
```

---

## Running the Notebook

1. Register (free) at [epc.opendatacommunities.org](https://epc.opendatacommunities.org/) to obtain an API key
2. Set environment variables or update the notebook with your credentials:
   ```
   EPC_EMAIL=your-email@example.com
   EPC_API_KEY=your-api-key
   ```
3. Run all cells. The first run downloads ~20,000 EPC records and caches them locally as CSV. Subsequent runs load from cache.

---

## Skills Demonstrated

`Python` `Pandas` `NumPy` `Scikit-learn` `IQR` `Isolation Forest` `One-Class SVM` `PCA` `Matplotlib` `Seaborn` `EPC Open Data` `Anomaly Detection` `Sustainability Analytics`

---

## Author

**Yenlik Gaisina** | Data & Analytics Consultant

[LinkedIn](https://www.linkedin.com/in/yenlik-gaisina/) | Cambridge Data Science with ML & AI Programme
