# 02 — Concentrate leftover-weight estimation (EfficientNet V2-S)

[← Research overview](../README.md) · [01 — Object detection](01_object_detection.md) · [03 — Grad-CAM](03_gradcam_interpretation.md)

## Objective, location and reference measurement

The regression task was to estimate **the mass (g) of concentrate remaining in an AMS feeder after a cow visit**, using the rectangular image crop selected by Mask R-CNN (see [section 01](01_object_detection.md)). The study was conducted at the University of Guelph's **Ontario Dairy Research Centre, Elora Research Station** in a free-traffic pen with one **DeLaval VMS V300** AMS. Cows were Holsteins of any parity; mean (± SD) pen stocking during the naturally occurring leftover collection period was **45.8 ± 1.7 cows**.

Programmed AMS concentrate allowances ranged from **1.75 to 9.75 kg/cow per day as fed**, dispensed at **500 g/min**. During a milking, the maximum delivery was 50% of the daily allotment, capped at **3.25 kg/visit**. Cows were eligible for a milking after a minimum **6 h** since the previous milking or when expected milk yield reached **9 kg**. These details describe the collection context, **not** additional predictor variables used by EfficientNet.

The concentrate was **tan-colored pellets**. Actual leftover weights were measured with a scale after the researchers collected the pellets and dust remaining in the feeder; the container's tare weight was subtracted. The procedure for obtaining reference weights differed between controlled (known-weight) and independent real-cow observations, described below.

## Datasets and sample acquisition

### Controlled known-weight observations: model development

Between **July 8 and October 4, 2026**, researchers prepared feed samples with known weights from **0 to 1,000 g in 50-g increments** (21 weight levels). For each level, **15 short video clips** were collected at different feeder locations: **315 clips / selected crops** in total.

For each sample, researchers weighed the feed, placed it in the AMS feeder, moved out of camera view, and repositioned the feed to capture different arrangements. Additional concentrate was added for subsequent known-weight levels. An image was selected from each clip using the feeder-detection procedure in [section 01](01_object_detection.md).

The 315 cropped images were partitioned into **252 training images (80%)** and **63 internal-validation images (20%)**, stratified by known leftover weight. A prior repository methods record documents a random split seed of **42**; this seed is *not stated in the supplied manuscript* and must be checked in the executed split script before being treated as a definitive reproducibility parameter.

### Naturally occurring leftovers: independent test

The independent dataset comprised **92 manually weighed cow-visit observations** from **March 4–April 24, 2026**, plus **17 clips on July 14, 2026** (included within the total of 92). Of all 92, **71 (77%)** corresponded to milkings between **08:00 and 12:00**, **13 (14%)** between **12:00 and 16:00**, and **8 (9%)** between **16:00 and 20:00**.

After a cow finished milking, the AMS was temporarily switched to manual mode. If the cow did not leave within 2 min, she was gently encouraged to exit. Researchers collected the leftover pellets and dust into a pre-weighed container, weighed the container with feed, and subtracted the tare weight. They recorded cow ID and the time the cow exited the AMS so that the corresponding camera footage could be matched to each reference weight.

**Independent observations were not used for training, early stopping or checkpoint selection.** However, all data were acquired in the same general AMS/camera setting, so the independent test evaluates different observations, not external-farm transferability.

| Dataset | Images | Reference-weight range | Role |
| --- | ---: | --- | --- |
| Controlled training | 252 | 0–1,000 g | Fit EfficientNet weights |
| Controlled internal validation | 63 | 0–1,000 g | Early stopping / checkpoint selection |
| Real-cow independent test (collected) | 92 | 1–1,459 g; median 14.5 g | Held-out evaluation before excluding two high-weight observations |
| Real-cow independent test (restricted analysis) | 90 | Excludes two observations above 800 g | Additional evaluation used for the reported restricted-test fit |

The manuscript describes two real-cow observations **above 800 g** as uncommon and removes them from the restricted independent-test analysis. It reports both the results with **all 92** and after excluding **2**, as detailed in the evaluation section. Such exclusions are an **evaluation/reporting decision**, not a training-data preprocessing step. Readers should retain the full-test results when assessing error across all observed weights.

## Video-to-crop processing and image dimensions

Original video was recorded at approximately **2,560 × 1,920 pixels, 20 frames/s**. The experiment timestamp was aligned to the video, a **4-s clip centered at the reference time (±2 s)** was extracted, and frames were sampled every **0.20 s**. Mask R-CNN localized the feeder. The highest-confidence detection among eligible candidates was used only if detector confidence was at least **0.90**. The corresponding **rectangular bounding box** was cropped and used as EfficientNet input. No mask-based removal of background pixels was performed.

Before resizing, the selected rectangular crops had the following dimensions:

| Dataset | Width (pixels) | Height (pixels) |
| --- | --- | --- |
| Controlled development (n = 315) | Median 426; range 397–466 | Median 445; range 426–467 |
| Independent test, standard-resolution crops (n = 90) | Median 413; range 389–469 | Median 433; range 409–467 |

Two of the **92 test crops** instead measured **188 × 197** and **194 × 203 pixels**, because the original video recordings were no longer available. **The manuscript does not establish whether these two lower-resolution crops are the same observations as the two >800-g exclusions; they should not be assumed to be identical.**

## Image preprocessing and augmentation

All selected crops were converted to **RGB**, resized to **224 × 224 pixels**, converted to tensors and normalized with the following **ImageNet** channel statistics:

| RGB channel | Mean | Standard deviation |
| --- | ---: | ---: |
| Red | 0.485 | 0.229 |
| Green | 0.456 | 0.224 |
| Blue | 0.406 | 0.225 |

**Training augmentation only** (n = 252): random horizontal flip (**probability 0.5**), random rotation (**within ±10°**) and random brightness (**0.2**), contrast (**0.2**), saturation (**0.2**) and hue (**0.03**) adjustments. The **internal-validation (n = 63)** and **independent-test (n = 92)** images underwent the same deterministic resizing and normalization **without random augmentation**.

## Model architecture and training

The prediction model was **EfficientNet V2-S** (Tan and Le, 2021), initialized with **ImageNet-1K pretrained weights**. The pretrained classification head was replaced with a **single-output fully connected regression layer**, returning a scalar for each selected crop.

Training targets were transformed from grams to the natural-log scale:

\[
y=\ln(1+w),\qquad w=\text{reference leftover weight in g}.
\]

The network minimized **mean squared error (MSE) on the transformed target**, rather than directly minimizing error in grams. The manuscript reports the following settings:

| Parameter | Value |
| --- | --- |
| Optimizer | AdamW |
| Initial learning rate | \(5\times10^{-5}\) |
| Weight decay | \(1\times10^{-4}\) |
| Batch size | 16 |
| Maximum training epochs | 200 |
| Early-stopping patience | 20 consecutive epochs |
| Minimum improvement in internal-validation loss | 0.0001 |
| Saved model | Checkpoint with the lowest internal-validation loss |

The model output \(\widehat{y}\) was converted back to grams by:

\[
\widehat{w}=\max\{0,\exp(\widehat{y})-1\}.
\]

Thus, a network output below zero after inverse transformation was clipped to **0 g**. This does **not** justify interpreting a failed feeder detection as zero leftover mass. The complete video clip was used to **select one crop**; the regression model predicted once from that crop, rather than averaging predictions across multiple frames.

Model development was performed in **Python 3.13.15** in **Google Colaboratory**, using an **NVIDIA Tesla T4 GPU**, according to the supplied manuscript. Its reported package versions were **PyTorch 2.11.0+cu130**, **Torchvision 0.26.0+cu130**, **Detectron2 0.6**, **scikit-learn 1.6.1**, **OpenCV 5.0.0.93**, **Pillow 11.3.0** and **NumPy 2.1.3**. These versions should be verified against the saved execution environment or lockfile before another researcher attempts to reconstruct it exactly.

## Prediction accuracy and reporting of exclusions

Metrics were computed from **back-transformed predictions in grams**, not transformed targets. For observations \(i=1,\ldots,n\), with reference weight \(w_i\), prediction \(\widehat{w}_i\) and mean reference weight \(\overline{w}\):

\[
\mathrm{RMSE}=\sqrt{\frac{1}{n}\sum_{i=1}^n(w_i-\widehat{w}_i)^2},
\qquad
R^2=1-\frac{\sum_{i=1}^n(w_i-\widehat{w}_i)^2}{\sum_{i=1}^n(w_i-\overline{w})^2}.
\]

| Evaluation subset | n | Reported \(R^2\) | Reported RMSE (g) |
| --- | ---: | ---: | ---: |
| Internal validation (known weights) | 63 | 0.93 | 77.4 |
| Independent real-cow test, all observations | 92 | 0.93 | 56.8 |
| Independent real-cow test, excluding two >800-g observations | 90 | 0.94 | 34.8 |

The manuscript reports that **76/92 independent records** had reference leftovers below 100 g and a median independent reference weight of **14.5 g**. Consequently, the independent-test data were heavily concentrated at low residual feed weights. This distribution matters when interpreting R² and RMSE. The two restricted-test exclusions should be disclosed whenever the **34.8 g RMSE** is quoted.

**Important manuscript check:** the current Figure 1B caption labels the independent test as **n = 92** while the text attributes the **0.94 / 34.8 g** result to the exclusion of two observations (**n = 90**). The figure legend, plotted points, analysis manifest and result table need to be reconciled before publication. The present method description does not silently change this inconsistency.

## Availability and limitations

The trained `EfficientNet.pth` checkpoint is supplied with the **Windows inference GUI** described in [`gui/README.md`](../gui/README.md). The documentation archive does **not** include the source data, original video, experimental record-to-clip manifest, training script, trained-model creation history or evaluation code. Therefore, the documentation supports **methodological transparency** but **cannot on its own reproduce model training and numerical metrics**.

The trained model was evaluated on one AMS installation and one pelleted-concentrate type. Generalization to other feeder geometries, concentrate types, cameras and farms remains untested in the manuscript.
