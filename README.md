# GBM Survival Prediction via Cellular Neighborhood Feature Engineering

> A computational biology pipeline that extracts phenotypic Cellular Neighborhood (CN) features from mass cytometry (CyTOF) data to improve survival prediction in Glioblastoma (GBM) patients.

---

## 📌 Project Overview

Glioblastoma (GBM) is the most aggressive primary brain tumor, with highly heterogeneous patient survival even under comparable treatments. Standard CyTOF-based survival analysis represents each patient by simple cell-type frequency vectors — capturing *what* cells are present, but not *how* they relate to each other.

This project proposes a **Cellular Neighborhood (CN) feature engineering approach** that constructs k-nearest-neighbor graphs over single-cell protein expression measurements, summarizes each patient's recurrent phenotypic neighborhoods, and uses these features for survival prediction.

**Dataset:** FlowRepository FR-FCM-Z24K — 28 primary GBM patients, 131,880 cells, 35 protein markers

---

## 👩‍💻 My Contributions

This was a 2-person team project. I was responsible for the following:

### 1. Data Loading
- Loaded and validated 28 FCS files from FlowRepository FR-FCM-Z24K
- Loaded clinical survival metadata (OS time, vital status)

### 2. Preprocessing
- Applied **arcsinh transformation** (cofactor=5) to compress highly skewed CyTOF marker intensities
- Normalized patient IDs across heterogeneous naming conventions (e.g. `LC10B` → `LC10`)
- Merged single-cell expression data with clinical survival outcomes

### 3. Cell Clustering
- Reduced 35D protein expression matrix to 20 principal components (PCA)
- Applied **Harmony batch correction** to remove cohort-driven variation across 3 patient batches (LC, RT, W)
- Ran **Leiden clustering** on corrected embeddings → 14 biologically distinct cell clusters
- Constructed **FlowSOM** cluster-frequency baseline feature matrix

### 4. Feature Engineering (Core Contribution)
- Built k-nearest-neighbor graphs in protein expression space (substituting physical distance with expression similarity)
- Computed each cell's local neighborhood composition profile
- Clustered neighborhood profiles into CN types via **MiniBatchKMeans**
- Aggregated per-patient CN frequencies into a patient-level feature matrix
- Conducted **grid search** over k ∈ {5,10,15,20} × n_CN ∈ {5,10,15,20} using LOO-CV C-index
- Optimal parameters: **k=15, n_CN=10**

---

## 📊 Key Results

CN-augmented features outperform both baselines across all survival metrics:

| Metric | FlowSOM Baseline | Leiden Baseline | CN-Augmented | vs. Leiden |
|---|---|---|---|---|
| C-index | 0.6497 | 0.6016 | **0.7005** | +16.4% |
| Integrated Brier Score | 0.1688 | 0.1860 | **0.1548** | -16.8% |
| AUC at 300 days | 0.5882 | 0.5508 | **0.6738** | +22.3% |
| AUC at 600 days | 0.7063 | 0.5813 | **0.6937** | +15.6% |

**Kaplan-Meier analysis:** CN-augmented risk scores produced statistically significant separation between high-risk and low-risk groups (log-rank p = 0.00238), compared to marginal separation with the Leiden baseline (p = 0.056).

---

## 📁 Project Structure

```
├── data/
│   ├── fcs_files/              # FCS files (not included — see Data section)
│   └── clinical_metadata.csv  # Clinical survival data (not included)
├── gbm_cytof_analysis.ipynb   # Full analysis pipeline
├── environment.yml            # Conda environment
└── README.md
```

> **Note:** Raw data not included due to patient privacy. Download FCS files from [FlowRepository FR-FCM-Z24K](https://flowrepository.org/id/FR-FCM-Z24K) and clinical metadata from the [RAPID study supplementary data](https://doi.org/10.7554/eLife.56879).

---

## ⚙️ Setup & Usage

### 1. Create conda environment
```bash
conda env create -f environment.yml
conda activate gbm_cytof
```

### 2. Prepare data
```bash
# Place FCS files in:
data/fcs_files/

# Place clinical metadata CSV in:
data/clinical_metadata.csv
```

### 3. Run analysis
Open and run `gbm_cytof_analysis.ipynb` in Jupyter:
```bash
jupyter notebook gbm_cytof_analysis.ipynb
```

---

## 🛠️ Tech Stack

- **Language:** Python 3.10
- **Single-cell analysis:** scanpy, anndata, harmonypy, flowsom
- **Survival modeling:** scikit-survival, lifelines
- **ML:** scikit-learn (KNN, MiniBatchKMeans)
- **Data processing:** fcsparser, numpy, pandas
- **Visualization:** matplotlib, seaborn

---

## 👥 Team

| Name | Role |
|---|---|
| Yewon Joung | Data loading, preprocessing, cell clustering, feature engineering |
| Yoolkyu Park | — |

*UNC Chapel Hill — COMP 683: Computational Biology (Spring 2026)*
