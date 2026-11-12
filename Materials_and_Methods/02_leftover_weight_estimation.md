# 02 — Concentrate leftover-weight estimation (EfficientNet V2-S)

[← Research overview](../README.md) · [01 — Object detection](01_object_detection.md) · [03 — Grad-CAM](03_gradcam_interpretation.md)

## Datasets and sample acquisition

### Controlled known-weight observations: model development

Rresearchers prepared feed samples with known weights from **0 to 1,000 g in 50-g increments** (21 weight levels). For each level, **15 short video clips** were collected at different feeder locations: **315 clips / selected crops** in total.

For each sample, researchers weighed the feed, placed it in the AMS feeder, moved out of camera view, and repositioned the feed to capture different arrangements. Additional concentrate was added for subsequent known-weight levels. An image was selected from each clip using the feeder-detection procedure in [section 01](01_object_detection.md).

The 315 cropped images were partitioned into **252 training images (80%)** and **63 internal-validation images (20%)**, stratified by known leftover weight.

## Video-to-crop processing and image dimensions

Original video was recorded at approximately **2,560 × 1,920 pixels, 20 frames/s**. The experiment timestamp was aligned to the video, a **4-s clip centered at the reference time (±2 s)** was extracted, and frames were sampled every **0.20 s**. Mask R-CNN localized the feeder. The highest-confidence detection among eligible candidates was used only if detector confidence was at least **0.90**. The corresponding **rectangular bounding box** was cropped and used as EfficientNet input. No mask-based removal of background pixels was performed.

Before resizing, the selected rectangular crops had the following dimensions:

| Dataset | Width (pixels) | Height (pixels) |
| --- | --- | --- |
| Controlled development (n = 315) | Median 426; range 397–466 | Median 445; range 426–467 |

## Image preprocessing and augmentation

All selected crops were converted to **RGB**, resized to **224 × 224 pixels**, converted to tensors and normalized with the following **ImageNet** channel statistics:

| RGB channel | Mean | Standard deviation |
| --- | ---: | ---: |
| Red | 0.485 | 0.229 |
| Green | 0.456 | 0.224 |
| Blue | 0.406 | 0.225 |

**Training augmentation only** (n = 252): random horizontal flip (**probability 0.5**), random rotation (**within ±10°**) and random brightness (**0.2**), contrast (**0.2**), saturation (**0.2**) and hue (**0.03**) adjustments. The **internal-validation (n = 63)** images underwent the same deterministic resizing and normalization **without random augmentation**.

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

## Availability and limitations

The trained `EfficientNet.pth` checkpoint is supplied with the **Windows inference GUI** described in [`gui/README.md`](../gui/README.md). The documentation archive does **not** include the source data, original video, experimental record-to-clip manifest, training script, trained-model creation history or evaluation code. Therefore, the documentation supports **methodological transparency** but **cannot on its own reproduce model training and numerical metrics**.
