# Manufacturing Control State Clustering — Semester Data Mining Project

## 1. Project Overview

This repository contains a university semester project for the **Knowledge Discovery / Data Mining** course (OZ, 2026). The project follows the **CRISP-DM** methodology and uses the dataset from Kaggle's [Tabular Playground Series — July 2022](https://www.kaggle.com/competitions/tabular-playground-series-jul-2022).

The Kaggle challenge is an **unsupervised clustering problem**: observations belong to unknown groups, but their true cluster labels and the number of clusters are not provided. The objective is to identify meaningful groups of observations using numerical features, compare clustering approaches, and interpret their results.

**Primary environment:** RapidMiner Studio. Python may be used for additional analyses, visualizations, or experiments when necessary. All processes and source files used in the final work must be submitted.

**Project status:** Initial dataset inspection completed; modeling and parameter optimization have not yet been completed. No final clustering results or performance scores are claimed in this README.

## 2. Dataset

**Source:** [Kaggle competition data](https://www.kaggle.com/competitions/tabular-playground-series-jul-2022/data)  
**Input file:** `data.csv`  
**Verified dataset shape:** 98,000 rows × 30 columns

| Attribute(s) | Description | Role |
| --- | --- | --- |
| `id` | Unique observation identifier (0–97,999) | Identifier; not a model feature |
| `f_00`–`f_06` | Seven continuous numerical features | Input features |
| `f_07`–`f_13` | Seven integer-valued, discrete numerical features | Input features |
| `f_14`–`f_21` | Eight continuous numerical features | Input features |
| `f_22`–`f_28` | Seven additional continuous numerical features | Input features |

There are **29 input features**, **no missing values** in the supplied `data.csv`, and **no ground-truth target/cluster column**. The feature names are anonymized: their physical meaning and units are not documented. Do not present any feature as a real temperature, pressure, cost, or machine measurement without evidence.

Initial inspection also found:

- `f_00`–`f_06` and `f_14`–`f_21` have means close to 0 and standard deviations close to 1.
- The integer features `f_07`–`f_13` have different scales and distributions.
- Some later continuous features (`f_22`–`f_28`) have greater variability and potentially different underlying structure.
- The strongest observed pairwise Pearson correlation was between `f_08` and `f_09`, approximately **−0.361**. No feature should be removed solely on this preliminary finding.

These are **observed properties of this file**, not proof of particular manufacturing states or an optimal clustering solution.

## 3. Problem Definition and Goals

### Business / scientific context

The project uses the hypothetical scenario of analyzing **simulated manufacturing control data** to distinguish operating states with different numerical patterns. A successful grouping could support monitoring, investigation of unusual states, or targeted diagnostics in a real system. Because the dataset is synthetic and anonymized, any operational interpretation remains illustrative.

### Data Mining goal

1. Explore the structure, distributions, and relationships of the 29 features.
2. Prepare consistent input data for distance-based and probabilistic methods.
3. Discover candidate clusters without using known labels.
4. Compare multiple algorithms and parameter configurations.
5. Describe and visualize the discovered groups and assess their stability.
6. Explain how an organization *might* use such a workflow and what additional domain evidence would be needed.

### Potential business indicators

Possible measures include time spent reviewing observations, the number of cases requiring manual investigation, and the cost of false alarms or missed anomalous states. **No actual monetary savings can be calculated from this dataset alone.** Economic benefits may be described with transparent hypothetical assumptions or formulas, not as measured results.

## 4. Academic Requirements

The semester assignment specifies three cumulative levels:

| Level | Maximum points | Required work |
| --- | ---: | --- |
| 1 | 10 | Complete CRISP-DM workflow; data analysis and preparation; at least two predictive/classification methods; compare and evaluate models |
| 2 | 15 | Level 1 plus an additional predictive method and hyperparameter optimization |
| 3 | 20 | Level 2 plus at least two descriptive methods, visualization, and interpretation |

**Important rubric mismatch:** The lecturer's generic requirements mention supervised classification/regression and metrics such as accuracy, confusion matrix, sensitivity, specificity, and ROC/AUC. The assigned Kaggle dataset is **unlabeled and designed for unsupervised clustering**. These supervised metrics cannot be calculated against genuine class labels from `data.csv`.

**Action required:** Confirm with the lecturer whether the predictive-model requirements can be replaced or adapted to unsupervised clustering. Until confirmed, **do not assume that three clustering algorithms automatically satisfy the supervised-model requirements**. Do not invent a target column or report pseudo-label accuracy as ground-truth accuracy.

## 5. Methodology: CRISP-DM

### Phase 1 — Business Understanding

- Define the analytical problem, scientific/business motivation, and expected outputs.
- Identify the input attributes (`f_00`–`f_28`) and intended output (a cluster assignment for each row).
- Define success criteria using appropriate unsupervised metrics and interpretable results.
- Explain any hypothetical business impact and limitations.

### Phase 2 — Data Understanding

- Import `data.csv` into RapidMiner with **comma** as the delimiter, **dot** as the decimal separator, and the first row as column names.
- Confirm row count, types, duplicates, missing values, unique values, and descriptive statistics.
- Use histograms and boxplots to examine the distributions of continuous and integer-valued features.
- Calculate a Pearson correlation matrix; consider non-linear dependence where appropriate.
- Document observations and include visual evidence in the report.

### Phase 3 — Data Preparation

- Assign the `id` attribute the special **id** role in RapidMiner and exclude it from model features.
- Check for missing or invalid values (none detected in the initial file inspection).
- Identify possible outliers; **do not remove them automatically**, because rare operating states may be important.
- Compare appropriate scaling strategies (e.g., **z-score standardization** and, if justified, robust scaling). Integer-valued attributes require consideration because their ranges differ from those of continuous features.
- Investigate redundant features and feature selection using evidence from EDA and model comparison; avoid arbitrary removal.
- Optionally use **PCA** for exploration, dimensionality reduction, and visualization. PCA by itself is not a clustering algorithm.
- Keep preprocessing consistent across model comparisons and document every transformation.

### Phase 4 — Modeling

Proposed methods (subject to instructor approval and practical performance):

| Method | Purpose | Parameters / experiments |
| --- | --- | --- |
| **K-Means** | Baseline partition-based clustering | Number of clusters `k`, initialization, random seed, restarts |
| **Gaussian Mixture Model (GMM)** | Probabilistic clustering | Number of components, covariance type, initialization, regularization |
| **DBSCAN** | Density-based clustering and noise detection | `eps`, `min_samples`, distance metric |
| **Hierarchical Clustering** | Additional descriptive analysis of cluster structure | Linkage method, distance metric, sample size |

Start with a **reproducible baseline**, then systematically tune parameters and compare alternatives. K-Means and GMM are priority methods; DBSCAN is a complementary approach. Hierarchical methods may need a representative subset because computing a full hierarchy for 98,000 observations is computationally expensive.

**Potential descriptive analyses:** cluster profiles (feature means/medians), cluster-size distributions, feature differences between groups, unusual or low-density observations, and interpretable rules where meaningful. For the full-credit descriptive requirement, identify **at least two actual descriptive modeling methods**; charts and PCA alone should not be counted as two models.

### Phase 5 — Evaluation

Because ground-truth labels are unavailable locally, focus on appropriate internal and stability-based validation:

- **Silhouette Score** — generally higher is better; specify the distance metric and whether sampling was used.
- **Davies–Bouldin Index** — lower is better.
- **Calinski–Harabasz Index** — higher is better.
- **Inertia / elbow analysis** — useful for comparing K-Means configurations, not a universal quality metric.
- **BIC/AIC** — useful for comparing suitable GMM configurations, subject to modeling assumptions.
- **Stability analysis** — inspect changes across seeds, subsamples, and preprocessing choices.
- **Cluster interpretability** — characterize sizes, distinctive features, and potential outliers.

Kaggle's official metric is the **Adjusted Rand Index (ARI)** against hidden labels. Without access to those labels, a local true-label ARI cannot be computed. Kaggle evaluation, if available, is an external reference rather than a substitute for documented local validation.

Do not directly compare metrics calculated on different feature spaces or radically different sampling conditions without stating those differences. For DBSCAN, explicitly report noise points and how they were handled during evaluation. Never fabricate scores: mark pending experiments as **Not run / TBD**.

Suggested experiment table:

| Model | Preprocessing | Main parameters | Silhouette | Davies–Bouldin | Calinski–Harabasz | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| K-Means | TBD | TBD | TBD | TBD | TBD | Not run |
| GMM | TBD | TBD | TBD | TBD | TBD | Not run |
| DBSCAN | TBD | TBD | TBD | TBD | TBD | Not run |

### Phase 6 — Deployment

No production deployment is required for the semester project. Describe a plausible workflow in which newly collected observations receive the same preprocessing and are assigned to previously defined clusters, then reviewed by domain specialists. Include monitoring, retraining, computational cost, and the limitations of synthetic/anonymized data. Note that not every algorithm (e.g., ordinary DBSCAN) supports straightforward assignment of new points without additional implementation.

## 6. Proposed Repository Structure

```text
.
├── README.md
├── data/
│   ├── README.md                 # Source and download instructions
│   └── .gitkeep                  # Do not assume the large CSV is committed
├── rapidminer/
│   ├── 01_data_understanding.rmp
│   ├── 02_data_preparation.rmp
│   ├── 03_kmeans.rmp
│   ├── 04_gmm.rmp
│   └── 05_additional_models.rmp
├── notebooks/                    # Optional Python experiments
├── results/
│   ├── figures/
│   ├── tables/
│   └── cluster_assignments/
├── docs/
│   └── semester_report.md
└── .gitignore
```

This is a **proposed structure**, not a claim that these files already exist. Keep raw data versioned only if permitted and appropriate; otherwise document how to download it from Kaggle. Save all RapidMiner `.rmp` processes and any Python code used to generate reported results.

## 7. Working Plan and Deliverables

1. **Baseline data audit:** import into RapidMiner, verify schema, and document data quality.
2. **EDA:** generate statistics, histograms, boxplots, and a correlation matrix; write evidence-based findings.
3. **Preprocessing:** handle data roles, scaling, and justified feature selection; save a reproducible pipeline.
4. **Baseline clustering:** run K-Means and GMM, documenting all settings.
5. **Optimization:** test candidate cluster counts and algorithm-specific parameters.
6. **Additional methods:** evaluate DBSCAN and a feasible second descriptive approach as required by the approved rubric.
7. **Evaluation:** prepare one consistent results table, cluster visualizations, profiles, and limitations.
8. **Documentation:** finalize the CRISP-DM report, deployment discussion, conclusions, and process/source-code deliverables.
9. **Defense:** be ready to explain the dataset, preprocessing choices, models, metrics, and findings.

Submission requirements from the lecturer: **written report plus all RapidMiner processes and/or source code**. Deadline: **end of teaching week 11**; defense: **week 12 during the practical class**. Confirm the calendar dates in MS Teams.

## 8. AI Assistant Context and Rules

When assisting with this repository:

- Treat this README as the **working project specification**, but prioritize real dataset inspection and the lecturer's explicit instructions if anything conflicts.
- Assume the main workflow is in **RapidMiner Studio**. Prefer exact operator names, port connections, parameter values, and actionable steps; use Python only when requested or genuinely helpful.
- Distinguish **verified facts**, **hypotheses**, **planned experiments**, and **completed results**.
- Do not assume a known number of clusters, a target label, meaningful physical feature names, or a particular best-performing algorithm.
- Do not invent numerical results, charts, repository files, completed tasks, or monetary savings.
- Preserve reproducibility: record data version, preprocessing choices, algorithm settings, random seeds, sampling methods, and evaluation conditions.
- Explain not only **what** to do but **why** each step supports the CRISP-DM report and the eventual defense.
- Flag the unresolved supervised-versus-unsupervised grading issue whenever recommending changes to the modeling plan.
- Keep documentation understandable to students and suitable for an English-language GitHub repository.

## 9. References

- [Kaggle — Tabular Playground Series, July 2022 (competition)](https://www.kaggle.com/competitions/tabular-playground-series-jul-2022)
- [Kaggle — Dataset files](https://www.kaggle.com/competitions/tabular-playground-series-jul-2022/data)
- CRISP-DM methodology (Business Understanding, Data Understanding, Data Preparation, Modeling, Evaluation, Deployment)

---

**Current next step:** complete and document the **Data Understanding** phase in RapidMiner before treating any clustering results as final.
