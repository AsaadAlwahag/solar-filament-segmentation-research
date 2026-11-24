# Learning guide: from Kaggle notebooks to understanding

## A. Four distinct outputs
- **Classification:** Does this image/crop contain a filament?
- **Detection:** Where is a candidate filament (bounding box)?
- **Semantic segmentation:** Which pixels belong to any filament?
- **Instance segmentation:** Which pixels belong to each different filament?

Our V6-B predicts semantic foreground and uses connected components for instances. **Two touching filaments may be merged; a weak gap may split one filament.**

## B. How to read the metrics
- **TP:** a predicted instance assigned to an expert instance at IoU ≥0.5 under one-to-one matching.
- **FP:** predicted instance unmatched after this procedure.
- **FN:** expert instance unmatched after this procedure.
- **IoU:** intersection area divided by union area for two masks.
- **PQ:** sum of IoUs of matched pairs / [TP + 0.5 FP + 0.5 FN].
- **Pixel Dice:** foreground pixel similarity, ignoring individual instance identity.

Example: A model can improve Dice and reduce PQ by splitting an already-correct filament into extra components (extra FP), or merging two objects (extra FN).

## C. What experiments taught us
1. **Checkpoint reproducibility:** frozen V6-B epoch 3 generated the original 10-image PQ .3302 when the restored evaluation reproduced the same rules.
2. **Threshold effects:** lower .35 gave smaller FP on 30 images, but also reduced TP; no threshold is 'best everywhere'.
3. **More epochs do not automatically generalize:** 30-image baseline PQ .3238 versus warm-start epoch-5 PQ .3142, even as Dice rose .6625→.6655.
4. **Boundary losses are not automatically instance-aware:** a boundary around a union of all filaments cannot indicate internal separation between touching instances.
5. **Error taxonomy needs matching:** a near-miss FP and its corresponding FN can arise from one single imperfect prediction.
6. **Dataset annotation policy matters:** different annotators may provide different instance sets for the same photograph.

## D. What we did not prove
- That thin or long filaments are missed more often (we measured their geometry but have not joined morphology to per-instance matches).
- That detection-based methods are inherently poor; V6-A training/proposal/mask reconstruction may have been the bottleneck.
- That attention is inferior; backbone is confounded.
- That a morphology-based mixture of experts is advantageous.
- That any of our local PQ scores are equivalent to an official Kaggle leaderboard score.

## E. How to turn this into rigorous follow-up
Freeze an untouched set; validate against the competition's official evaluator; export per-instance matches; use matched training seeds for ablations; manually inspect representative merges/splits; compare a general model against a spine-guided approach. Do not use test/diagnostic images repeatedly to choose hyperparameters.
