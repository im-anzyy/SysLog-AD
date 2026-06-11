# Log-Based Anomaly Detection using Machine Learning

**Research project** comparing unsupervised ML algorithms for detecting anomalous behavior in real-world system logs at scale.

---

## The Problem

Production systems generate millions of log lines daily. Security teams cannot manually inspect them. Can unsupervised ML reliably detect anomalous sessions — with no labeled training data — at production scale?

## What Makes This Different

Most anomaly detection tutorials use tiny datasets and supervised learning. This project uses:
- **11.1M real log lines** from Hadoop (not a sample CSV)
- **Unsupervised methods only** — as required in real SOC deployments
- **575,061 sessions** evaluated end-to-end
- **Honest negative results** — LOF and DBSCAN failed, and we explain why

## Results

| Model | Precision | Recall | F1 | ROC-AUC | FPR |
|---|---|---|---|---|---|
| **Isolation Forest** | 0.6219 | 0.6223 | 0.6221 | **0.9609** | 0.0114 |
| One-Class SVM | 0.5740 | **0.7468** | **0.6491** | 0.9246 | 0.0167 |
| Local Outlier Factor | 0.1455 | 0.1530 | 0.1492 | 0.5924 | 0.0271 |
| DBSCAN | 0.6072 | 0.3346 | 0.4315 | 0.5000 | 0.0065 |

**Key finding:** Isolation Forest dominates on ROC-AUC (0.96). One-Class SVM is superior when recall matters (catches 74.7% of anomalies). Density-based methods fail on high-dimensional session feature spaces.

## Pipeline
Raw HDFS Logs (11.1M lines)
|
| regex parsing
v
Structured DataFrame (Date, Time, PID, Level, Component, Content)
|
| group by Block ID
v
575,061 Sessions
|
| feature engineering
v
19-dimensional Feature Matrix
|
| StandardScaler
v
4 Unsupervised ML Models --> Evaluation & Comparison


## Key Research Findings

1. **PacketResponder activity** is the strongest anomaly signal (discriminability score 3.55) — consistent with HDFS block replication failure patterns
2. **Anomalies are not uniform** — PCA reveals two subtypes: obvious outliers and subtle behavioral deviations mixed within normal data
3. **Algorithm scalability matters** — LOF and DBSCAN required subsampling at 575K sessions due to O(n²) complexity, directly impacting evaluation quality
4. **Score separation** — Isolation Forest produces the clearest normal/anomaly score distributions of all four models

## Notebooks

| # | Notebook | Description |
|---|---|---|
| 01 | verify_setup | Environment and dataset validation |
| 02 | data_exploration_preprocessing | Parsing 11M logs, label analysis |
| 03 | feature_engineering | 19-feature session pipeline |
| 04 | model_training_evaluation | Training and evaluating 4 models |
| 05 | visualization_analysis | PCA, t-SNE, ROC curves, heatmaps |
| 06 | research_paper_draft | Findings and paper draft |

## Dataset

- **Source:** HDFS_v1 from [LogHub](https://zenodo.org/records/8196385)
- **Raw logs:** 11,175,629 lines
- **Sessions:** 575,061 unique Block ID sessions
- **Labels:** 558,223 Normal (97.07%) | 16,838 Anomaly (2.93%)

## Reproduce This Project

```bash
pip install numpy pandas scikit-learn matplotlib seaborn jupyter scipy
git clone https://github.com/im-anzyy/SysLog-AD
cd log-anomaly-detection
jupyter notebook
```

Run notebooks 01 through 06 in order.

## Limitations and Future Work

- LOF and DBSCAN evaluated on subsamples due to quadratic time complexity
- Session features treat logs as bag-of-events — temporal order not preserved
- **Future:** LSTM autoencoders for sequence-aware anomaly detection
- **Future:** Ensemble combining Isolation Forest and One-Class SVM
- **Future:** Real-time streaming detection for live SOC deployment

## Tech Stack

Python | Jupyter | scikit-learn | pandas | NumPy | Matplotlib | Seaborn
