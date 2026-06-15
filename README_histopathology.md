# Deep Learning for Prostate Histopathology

A compact computational pathology pipeline for prostate tissue patch classification, grade progression modelling, and structural interpretation.

The project trains an EfficientNet-B2 classifier on H&E prostate biopsy patches and analyses the learned feature space to understand how tissue morphology changes across Gleason grades.

## What it does

- Classifies prostate tissue patches into five classes:
  `Stroma`, `Normal`, `Gleason 3`, `Gleason 4`, `Gleason 5`
- Extracts deep feature embeddings from the trained model
- Maps morphology progression from Gleason 3 to Gleason 5
- Detects uncertain or transitional patches, especially around Gleason 4
- Tests cross-domain generalization across independent imaging cohorts
- Computes interpretable structural features from tissue architecture
- Introduces a Structural Accessibility Index to estimate how open or compact a tissue patch is

## Core idea

The model does more than classify patches.

It learns a latent morphology space where prostate cancer grades form a biological progression: Gleason 3 and Gleason 5 sit at opposite ends, while Gleason 4 behaves as a heterogeneous transition zone.

## Model

- Backbone: EfficientNet-B2
- Input: H&E image patches
- Patch size: resized to 224 × 224
- Output classes: 5
- Optimizer: AdamW
- Scheduler: warmup + cosine decay
- Main metric: macro F1

## Results

| Setting | Macro F1 |
|---|---:|
| Cohort A test | 0.944 |
| Cohort B test | 0.953 |
| Cohort A → Cohort B | 0.396 |
| Cohort B → Cohort A | 0.558 |
| Cohort A → NTU external cohort | 0.457 |

The model performs strongly in-domain but fails under domain shift. This is the main finding: high test accuracy inside one cohort does not mean clinical robustness across sites.

## Key findings

Gleason 4 is the hardest class because it is not one clean morphology. It spans multiple tissue architectures and sits between lower-grade and higher-grade disease.

Stain normalization alone did not fix external performance. The failure appears deeper than color shift.

The model remained confident even when wrong on the external NTU cohort, which is a serious deployment risk.

Structural features such as lumen fraction, nuclei fraction, texture contrast, and tissue homogeneity align with grade progression. Higher-grade tissue becomes denser, less glandular, and structurally less accessible.

## Repository structure

```text
.
├── data/                  # Dataset folders or symlinks
├── notebooks/             # Exploration and analysis notebooks
├── src/
│   ├── train.py           # Model training
│   ├── evaluate.py        # Test and cross-domain evaluation
│   ├── embeddings.py      # Feature extraction and UMAP analysis
│   ├── structural.py      # Structural feature extraction
│   └── utils.py
├── results/
│   ├── figures/
│   ├── metrics/
│   └── checkpoints/
├── requirements.txt
└── README.md
```

## Quick start

```bash
git clone <repo-url>
cd <repo-name>

python -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt
```

Train:

```bash
python src/train.py --config configs/cohort_a.yaml
```

Evaluate:

```bash
python src/evaluate.py --checkpoint results/checkpoints/best.pt --test_dir data/cohort_a/test
```

Extract embeddings:

```bash
python src/embeddings.py --checkpoint results/checkpoints/best.pt --data_dir data/all_test
```

## Notes

This repository is intended for research and reproducibility, not clinical deployment.

The strongest result is not the classifier score. The strongest result is the failure mode: domain shift can produce confident wrong predictions, even when in-domain performance looks excellent.

## Authors

Rabin Bajagain  
Thian Yu Wen Joanne

Precision Oncology, April 2026
