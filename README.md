# Alzheimer's Cascading Multimodal Pipeline

A cost-aware, multi-stage machine learning prototype that helps **prioritise patients for Alzheimer's diagnostic escalation**. Instead of sending everyone straight to expensive MRI or PET scans, the pipeline orders investigations from cheap to costly and escalates a patient to the next stage only when the model-estimated risk crosses a threshold.

> **Disclaimer:** This is a research prototype. It is **not** a diagnostic tool, has not been clinically validated, and must not be used to make medical decisions.

---

## Idea

```
All patients
    |
Stage 1: Cognitive screening (AGE, PTGENDER, MMSE, MoCA, CDR-SB)
    |  risk >= 40%?  -- no --> routine follow-up
    v
Stage 2: Biomarkers (+ APOE4, ABETA, TAU, PTAU)
    |  risk >= 50%?  -- no --> monitor
    v
Stage 3: Structural MRI (+ Hippocampus, WholeBrain, Ventricles, Entorhinal, ICV-normalised)
    |  risk >= 75% -> high priority
    v
Stage 4: PET (FDG, AV45) -- design stage, not modelled yet
    |
Neurologist action list
```

The thresholds (40% / 50% / 75%) are prototype design parameters, **not** clinically derived cut-offs.

## Data

- **ADNIMERGE** table (version dated 5 March 2026), loaded through KaggleHub.
- Baseline visits only (`VISCODE == 'bl'`): 2,430 records, 2,419 after removing missing diagnoses.
- Target `High_Risk`: 0 = cognitively normal; 1 = EMCI, LMCI, AD or SMC (about 77.7% positive).

ADNI data is subject to the ADNI data use agreement. Please check that your use complies with it. This repository does not redistribute the data.

## Results (test set, n = 484)

| Stage | Best model | Accuracy | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| 1: Cognitive | XGBoost | 0.866 | 0.923 | 0.914 | 0.921 |
| 2: + Biomarkers | XGBoost | 0.847 | 0.894 | 0.901 | 0.925 |
| 3: + MRI | XGBoost (accuracy) / Random Forest (AUC 0.940) | 0.864 | 0.915 | 0.912 | 0.931 |

Random Forest, XGBoost and an MLP were compared at each stage. A model that predicts "high risk" for everyone would score about 0.777 accuracy. The main takeaway is that the cheap Stage 1 inputs already carry most of the signal, and later stages add modest gains.

## What is in this repository

| File | Description |
|---|---|
| `Alzheimers_Cascading_Multimodal_Pipeline.ipynb` | Full pipeline: preprocessing, EDA, stage-wise models, escalation engine and report generator |
| `LICENSE` | MIT License |

The `AlzheimersMultiStageScreeningEngine` class runs a patient through the stages and produces a readable clinical-style report.

## How to run

1. Open the notebook in [Google Colab](https://colab.research.google.com/) (upload it, or use "Open in Colab" from GitHub).
2. Run all cells. The data is downloaded automatically via `kagglehub`.
3. Or run locally:

```bash
pip install pandas numpy scikit-learn xgboost matplotlib seaborn missingno kagglehub
jupyter notebook Alzheimers_Cascading_Multimodal_Pipeline.ipynb
```

## Known limitations

- **Label circularity:** ADNI baseline diagnosis is partly defined using MMSE and CDR, so Stage 1 performance is likely optimistic.
- **Imputation before splitting:** KNN imputation was fitted on the full dataset, and about 50% of biomarker values were imputed. Fit imputers on training data only for a stricter evaluation.
- **Biomarker source:** ABETA, TAU and PTAU in ADNIMERGE are CSF-derived, not plasma. Some report text in the notebook says "plasma" and should be read accordingly.
- **Probabilities are not calibrated**, and thresholds are not tuned.
- **Single 80/20 split**, default hyperparameters, no confidence intervals and no external validation.
- **Stage 4 (PET) has no trained model**, and no explainability (SHAP) is included yet.

## Roadmap

- [ ] Fit all preprocessing inside cross-validation folds
- [ ] Use a longitudinal progression label instead of the baseline diagnosis
- [ ] Calibrate probabilities and tune thresholds with a cost-benefit analysis
- [ ] Add SHAP explanations to each report
- [ ] Build the PET stage (FDG, AV45)
- [ ] External validation (e.g. OASIS-3) and plasma p-tau217 as the Stage 2 input

## Paper

A research paper describing this work is available on Zenodo: *DOI link to be added after upload.*

## Citation

If you use this work, please cite:

```
Sanidhya. A Cost-Aware, Four-Stage Cascading Machine Learning Framework for
Prioritizing Patients for Alzheimer's Disease Diagnostic Escalation: Evidence
from the ADNI Baseline Cohort. Bikaner Technical University, 2026.
```

## Acknowledgements

Sincere thanks to **Prof. Neeraj Chaudhary** for her guidance and encouragement, to **Bikaner Technical University**, and to the **Alzheimer's Disease Neuroimaging Initiative (ADNI)** for the data.

## License

Released under the [MIT License](LICENSE).
