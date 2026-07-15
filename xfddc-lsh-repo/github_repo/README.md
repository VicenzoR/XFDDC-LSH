# XFDDC-LSH

**An Explainable Framework for Fault Detection, Diagnosis & Correction in Industrial Processes**

Bachelor thesis in Industrial Engineering — Universidade Federal do Espírito Santo (UFES).
Advisor: Prof. Dr. Renato A. Krohling. Defended July 9, 2026.

## What this is

XFDDC-LSH is a hierarchical framework for monitoring continuous chemical processes that combines:

- **XGBoost cascade** — a binary global anomaly detector that triggers a committee of 17 fault-specific specialist classifiers (One-vs-Rest)
- **LSH (Locality-Sensitive Hashing)** — geometric data distillation that cuts redundant training samples by ~90% with minimal accuracy loss
- **EWMA** — exponential smoothing on the detector's output to suppress false alarms from transient sensor noise
- **TOPSIS** — multi-criteria decision ranking that arbitrates between competing specialist diagnoses using both real-time probability and each specialist's validation track record
- **SHAP + DiCE** — local explainability (which sensors drove the diagnosis) and counterfactual prescription (which actuator adjustments would return the plant to normal operation)

Validated on the **Tennessee Eastman Process (TEP)** benchmark (Downs & Vogel, 1993; extended simulation by Rieth et al., 2017) across 18 fault classes and 8.64M test samples.

## Results

| Metric | Baseline | LSH-distilled |
|---|---|---|
| Global Accuracy | 92.78% | 90.66% |
| Cohen's Kappa | 0.921 | 0.898 |
| Fault Detection Rate (macro) | 92.36% | 89.74% |
| False Alarm Rate (macro) | 0.46% | 0.59% |
| F1-Score (macro) | 93.21% | 90.90% |

The LSH-distilled variant trades ~3.2 points of diagnostic performance for a >50% reduction in training time and data volume — see the thesis for the full ablation and per-class breakdown.

## Repository structure

```
notebooks/
  Gerador_joblib-ts_final.ipynb    # trains the baseline (full-data) specialist committee
  Gerador_LSH-JOBLIB_FINAL.ipynb   # trains the LSH-distilled specialist committee
  detector_global_ts.ipynb         # trains + calibrates the baseline global detector
  detector_lsh_final.ipynb         # LSH distillation step + LSH-variant global detector
  teste-ts-final.ipynb             # full online evaluation over the test set (baseline)
  teste-lsh-final.ipynb            # full online evaluation over the test set (LSH variant)
  topsis_shap_dice.ipynb           # single-sample case study: TOPSIS + SHAP + DiCE walkthrough
data/            # place the TEP dataset here (see below) — not included in this repo
models/
  baseline/      # .joblib artifacts produced by the *_ts_* notebooks
  lsh/           # .joblib artifacts produced by the *_LSH_* notebooks
```

## Getting the data

The TEP dataset used here is the extended simulation by Rieth et al. (2017), publicly available on Harvard Dataverse:
https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/6C3JR1

Download the `.RData` files and place them under `data/`:
`TEP_FaultFree_Training.RData`, `TEP_Faulty_Training.RData`, `TEP_FaultFree_Testing.RData`, `TEP_Faulty_Testing.RData`.

## Running it

1. `pip install -r requirements.txt`
2. Download the TEP dataset into `data/` (above).
3. Run the training notebooks first (`Gerador_joblib-ts_final.ipynb` and/or `Gerador_LSH-JOBLIB_FINAL.ipynb`, `detector_global_ts.ipynb` and/or `detector_lsh_final.ipynb`) — these populate `models/`.
4. Run `teste-ts-final.ipynb` / `teste-lsh-final.ipynb` for the full evaluation of each variant, or `topsis_shap_dice.ipynb` for the single-sample explainability walkthrough.

Training the full committee on the complete TEP dataset is computationally heavy (millions of samples, 17 specialist models with Optuna hyperparameter search); a GPU is recommended but not required.

## Notes

- Package versions weren't pinned in the original research environment; `requirements.txt` lists the required packages without strict version constraints. If you hit an incompatibility, `xgboost>=2.0` and `shap>=0.44` are known to work.
- This code accompanies an academic thesis rather than a production system — clarity and reproducibility of the reported results were prioritized over engineering hardening.

## Citation

If you use this work, please cite the thesis (De Nadai, V. R. *XFDDC-LSH: an explainable framework for fault detection, diagnosis and correction in industrial processes*. UFES, 2026, advisor: R. A. Krohling).

## License

MIT — see [LICENSE](LICENSE).
