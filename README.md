# Solar Filament Segmentation — Comparative Deep Learning and Error Analysis

**Status:** Exploratory research archived November 2026. **No claim of a novel validated model or leaderboard-level result.**

This project examines instance segmentation of solar filaments in full-disk GONG H-alpha observations from the Kaggle Solar Filament Segmentation Challenge 2026. It compares three inspired architectures, evaluates the distinction between foreground-mask accuracy and instance Panoptic Quality (PQ), and analyses filament geometry and failure modes.

## Research question

Why do improved filament-pixel predictions not necessarily translate into better individual-filament instance reconstruction?

## Data and protocol

- Competition subset: 707 unique 2048 × 2048 photographs, 1,154 image/expert annotation records, 8,199 annotated filament records.
- Deterministic photograph-level 565/142 train/validation split (`random_state=42`), one expert record selected per photograph (most annotations; record-ID tie-break).
- Initial cross-architecture comparison: **10 selected full-resolution validation photographs**, 92 GT instances (exploratory and reused for model and threshold selection).
- Additional fixed diagnostic subset: **30 other validation photographs**, 258 selected GT instances (also used in subsequent diagnostic analyses).
- Local simplified instance-PQ calculation: one-to-one matches at mask IoU ≥ 0.50, `PQ = sum(matched IoUs)/(TP + 0.5 FP + 0.5 FN)`. These are **not verified official competition scores**.

## Architecture comparison (10-image exploratory set)

| Model | Approach | TP | FP | FN | Local PQ |
|---|---|---:|---:|---:|---:|
| V6-A | YOLOX-Tiny + box-conditioned U-Net, mask merging | 44 | 147 | 48 | 0.1998 |
| V6-B epoch 3 | ImageNet-pretrained ConvNeXt-Tiny + U-Net | 49 | 53 | 43 | **0.3302** |
| V6-C epoch 3 | ResNet-34 + spatial attention-gated U-Net | 49 | 68 | 43 | 0.2998 |

This is an **architecture-level** comparison, not a controlled isolation of attention; backbones, model sizes and training histories differ.

## V6-B follow-up experiments

- A segmentation threshold of **0.35** versus **0.55** improved exploratory 10-image PQ from 0.3302 to 0.3713, but on a separate 30-image diagnostic subset improved PQ only from **0.3238 to 0.3319**. At 0.35, TP decreased 136→131 while FP decreased 160→135: the gain was not a universal recall improvement.
- Warm-start BCE+Dice training from epoch 3 to epoch 5 increased 10-image PQ from 0.3302 to **0.3509** but decreased 30-image PQ from **0.3238 to 0.3142** (FP 160→182, TP unchanged at 136).
- On those 30 images, threshold-0.55 foreground Dice increased slightly **0.6625→0.6655** while instance PQ declined **0.3238→0.3142**.
- On the original 10 images, one epoch of exploratory boundary-weighted BCE+Dice gave **PQ 0.2921**, versus **0.3430** for an unmatched-random-seed ordinary BCE+Dice warm start. This does **not** establish boundary supervision is universally ineffective.
- Of the 160 unmatched V6-B epoch-3 predictions on the 30-image subset: 59 had no selected GT overlap; 53 had best individual GT IoU between 0 and 0.30; 48 had best IoU 0.30–0.49. These groups describe unmatched **predictions**, not disjoint causal explanations for the 122 false-negative GT objects.

## Morphology findings

All 8,199 annotation records have parseable spine coordinate sequences. Across all annotation records: median spine length ≈111 px, effective width (`polygon area / spine length`) ≈11.08 px, median elongation ≈10.51. Only 80/8,199 records (1.0%) have a bounding-box dimension >512 px; this **does not rule out cropping truncation**.

On the 30-image set, 258 selected annotations have median spine length ≈115.79 px, effective width ≈10.55 px, tortuosity ≈1.058. Morphology **has not yet been joined with per-filament detection outcomes**, so morphology-based specialist models remain a hypothesis.

## Main limitations

1. Repeated exploratory use of the same validation images; no final untouched test-set verification.
2. Ground truth derived from one of several expert records, not an inter-annotator-consensus analysis.
3. Full notebooks, executable pipelines and trained weights are **not included in this archive** until separately exported from Kaggle.
4. Model comparisons use different backbones/training details, and warm-start comparisons are not random-seed matched.
5. Pixel foreground Dice and local PQ measure different tasks; neither alone proves a causal mechanism.

## Repository structure

- `docs/RESEARCH_REPORT.md`: detailed study and lessons learned
- `docs/LEARNING_GUIDE.md`: concepts explained and how to interpret metrics
- `docs/REPRODUCIBILITY.md`: how to recover/export the Kaggle notebooks and models
- `results/`: machine-readable aggregate results
- `poster/ABSTRACT_DRAFT.md`: non-archival workshop extended abstract draft
- `poster/POSTER_PLAN.md`: figure and section layout
- `REFERENCES.md`: verified scientific sources
- `notebooks/`: placeholder/instructions for importing actual Kaggle notebook exports

## Data access

Data: https://www.kaggle.com/competitions/filament-segmentation-2026 . Source MAGFiLO paper: https://doi.org/10.1038/s41597-024-03876-y . **Do not commit competition images, private Kaggle assets or large model checkpoint files without checking their reuse terms.**

## Publication status

Research documentation for educational and reproducibility purposes; not a peer-reviewed paper. Workshop abstract is a **draft**, not submitted or accepted. Authors/affiliations require confirmation before public release.


## Uploaded notebook source archive (24 November 2026)

Five distinct Kaggle notebook exports are included under [`notebooks/`](notebooks/), with output data stripped for repository portability. See [`docs/NOTEBOOK_PROVENANCE.md`](docs/NOTEBOOK_PROVENANCE.md). The notebooks have **not** been validated as an end-to-end clean rerun; final competition submission code and the separate morphology notebook still require integration and verification. No competition dataset images or model checkpoints are redistributed.
