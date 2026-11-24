# Research report — What we tested, what we learned

## 1. Motivation
Solar H-alpha filament segmentation requires separating distinct elongated, sometimes low-contrast features. The competition evaluates **individual instance masks**, whereas many U-Net variants natively predict one binary foreground mask. Pixel connectivity, mask quality and object identity are not interchangeable.

## 2. Datasets and evaluation
Our Kaggle subset consists of 707 unique solar photographs (565 training; 142 validation), 1,154 expert-record entries and 8,199 annotation entries. Some photographs have multiple annotation records (411 with one; 145 with two; 151 with three). We selected one record per photograph by maximum annotation count, followed by ascending ID, which is a modelling choice and **not an adjudicated consensus**.

10 full images (92 selected GT instances) were used to compare model families. An additional 30 images (258 GT instances) served as a development/diagnostic sample. Several choices, including area and thresholds, were informed by evaluation data. No independent official/leaderboard comparison is asserted.

## 3. Model families

### V6-A: detection and box-conditioned instance masks
Sartorius-inspired YOLOX-Tiny detector plus a box-conditioned U-Net. A local selected 20-tile pilot gave PQ 0.3808; on 10 complete images, merged output produced TP44 FP147 FN48 PQ0.1998. This is strong evidence that the **specific end-to-end pipeline** generated many unmatched instances at full resolution, not proof all detector-guided instance segmenters fail.

### V6-B: ConvNeXt-Tiny U-Net
~31.9M parameters. Encoder pretrained on ImageNet; binary segmentation with BCE-with-logits + soft Dice, 512×512 cropped training, 512 stride-256 tiled full-image inference. Instance masks extracted by thresholding, 3×3 open/close, connected components and area/confidence filters. Epoch-3 checkpoint is main reference. V6-B had the strongest 10-image exploratory PQ (0.3302).

### V6-C: ResNet-34 attention-gated U-Net
~24.7M parameters. Spatial attention on skip features; three epochs. At the shared evaluation configuration: TP49 FP68 FN43 PQ0.2998. We cannot infer whether attention itself helps or harms because the encoder differs from V6-B.

## 4. Postprocessing diagnostics
One 15-GT-image analysis found morphology changed net thresholded pixel count only slightly (27,741 threshold pixels to 27,702 after open/close), while component filtering removed ~3,071 more pixels. Pixel-level filament coverage was variable; one GT filament went from 33.6% threshold coverage to 0% final retained coverage. This illustrates why raw foreground response and reconstructed instances must be evaluated separately.

Across 10 images: threshold .55 PQ .3302 and threshold .35 PQ .3713. Across 30 other images: .55 produced TP136 FP160 FN122 PQ .3238; .35 produced TP131 FP135 FN127 PQ .3319. Lower threshold therefore improved aggregate PQ slightly but **did not improve the number of matched instances** on the larger diagnostic sample.

## 5. Additional training and loss experiments
A warm-start epoch-4 BCE+Dice run achieved 10-image PQ .3430; epoch-5 achieved .3509. But on 30 images, epoch-5 PQ was .3142 vs baseline .3238, with 22 additional false positives and identical TP136. Pixel Dice went .6625 to .6655 under the same threshold and image set. Loss optimization/pixel overlap does not guarantee instance quality.

An exploratory one-epoch boundary-weighted BCE+Dice run (same V6-B architecture, differing random crop draws) produced 10-image PQ .2921 vs conventional warm-start epoch-4 .3430. The loss used foreground-union-derived boundaries, not per-instance contact boundaries. Do not overinterpret this as a general failure of boundary supervision.

## 6. False-positive analysis
On 30 images, baseline epoch-3 model produced 160 unmatched predicted instances: 59 no intersection with any selected GT mask; 53 positive intersection but best individual IoU <.30; 48 best individual IoU .30–.49. A predicted-mask union overlap above 50% does not imply any single-object IoU above 50%, particularly when masks merge or split. We did not visually classify all these errors into causal mechanisms.

False-negative population: 122 GT instances unmatched; **mechanism-level breakdown unknown**. It would be invalid to simply label all 122 as 'not detected'.

## 7. Filament morphology
All 8,199 annotation entries had usable spine polylines. Approximate width is polygon area divided by spine arc length; tortuosity is arc length over endpoint distance. Median from all annotations: length 111.27 px, effective width 11.08 px, elongation 10.51. 1% of annotation bounding boxes exceed 512 px max extent, but random crop positioning may still truncate instances. On 258 diagnostic GT instances: median length 115.793, width 10.545, tortuosity 1.058. Linking each of these with instance matching remains **unfinished**.

## 8. Main conclusion
The evidence supports a problem formulation distinction: a foreground segmentation objective may improve pixel-wise mask overlap while end-to-end connected-component instance PQ deteriorates. Post-processing choices interact with object identity and confidence filtering. A future model could explicitly learn instance geometry/spines or use morphology-aware routing, but neither concept has yet been validated here.

## 9. Scientific integrity / archive status
These were exploratory Kaggle notebook runs, not a publication-grade controlled benchmark. No claim of novelty, winning leaderboard rank, official PQ equivalence, statistical significance, or independent replication is made.
