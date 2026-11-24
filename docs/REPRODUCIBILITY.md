# Reproducibility and Kaggle export checklist

These documents reconstruct **the experimental narrative**, not the full training source code. The real Kaggle notebooks must be added separately; do not label this repo executable before doing so.

## Source notebook exports to add
1. `Solar_Filament_V6_Architecture_Research` — V6-A YOLOX/box U-Net notebook.
2. `Solar_Filament_V6B_SenNet_Architecture` — V6-B ConvNeXt model, diagnostics and continuation experiments.
3. `Solar_Filament_V6C_Attention_Architecture` — V6-C attention network.
4. `Solar_Filament_Dataset_Morphology_and_Error_Analysis` — morphology and expert annotation investigations.
5. The earlier V5-B baseline notebook (if redistributable).

From Kaggle, open each saved notebook version and download/export the `.ipynb` as `notebooks/<corresponding_name>.ipynb`. Remove secret tokens and private paths before publishing. Preserve version links in README after verifying real URLs.

## Persistent model assets
- V6-B epoch-3 frozen checkpoint: Kaggle private dataset `asaadwahag/v6b-final-research/v6b_convnext_unet_best.pth` (121.82 MiB observed).
- V6-B+1 / +2 epoch checkpoints were in an archived saved notebook output; their publication is optional.
- V6-C epoch-3 checkpoint recorded as saved in its Kaggle notebook.
- V6-A stronger 300-step YOLOX/320-step U-Net checkpoint persistence requires checking before making reproduction claims.

Do not commit 100+ MB `.pth` weights to ordinary Git; consider Git LFS or Kaggle-hosted model assets subject to permissions. Do not reupload competition images without license review.

## Requirements to reproduce the results
- Original competition train images and MAGFiLO JSON annotations.
- Correct unique-photo split 565/142 and deterministic record selection.
- Network classes and precise preprocessing from the exported notebooks.
- IoU≥0.5 one-to-one instance matching and full-resolution inference with overlapping tiles.
- Exactly fixed post-processing settings for each comparison.

A complete clean-environment reproduction has **not** yet been performed.
