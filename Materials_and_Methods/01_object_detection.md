# 01 — Feeder detection and image extraction (Mask R-CNN)

[← Research overview](../README.md) · [02 — Leftover-weight estimation](02_leftover_weight_estimation.md) · [03 — Grad-CAM](03_gradcam_interpretation.md)

## Objective and relationship to the regression model

The object-detection stage locates the **concentrate feeder** in a full AMS camera image or a sampled video frame. It produces a **rectangular bounding-box crop** for the subsequent EfficientNet V2-S regression model. Mask R-CNN was trained as an instance-segmentation model, but **its predicted segmentation mask was not used to remove background pixels from the regression input**. Consequently, visible bowl edges and any other content within the selected bounding box remain in the crop. Detection confidence concerns feeder localization, not confidence in the predicted leftover mass.

## Study setting and imaging equipment

The work was conducted at the Ontario Dairy Research Centre, University of Guelph Elora Research Station (Elora, Ontario, Canada), using a DeLaval VMS V300 automated milking system (AMS). A **Reolink RLC-511W** camera was mounted on the ceiling of the AMS room, approximately **2.3 m from the feeder**. The original on-site video had an approximate resolution of **2,560 × 1,920 pixels** and a frame rate of **20 frames/s**.

These original-video specifications differ from the **screenshots used to train the feeder detector**: 50 PNG screenshots, obtained at various times of day between **November 11 and 17, 2025**, had an average image size of approximately **1,376 × 1,032 pixels**. Lighting conditions and the amount of leftover feed varied across the screenshots.

## Annotations, dataset partition and model initialization

Each of the **50 screenshots** was manually annotated in **LabelMe**. The feeder was traced as a **closed polygon** and labeled as the sole foreground class, `feeder`. The per-image LabelMe JSON annotations were converted to **COCO instance-segmentation format**; the resulting annotation JSON and associated image files were registered in Detectron2.

- **Training:** 30 annotated images.
- **Independent detector test:** 20 held-out annotated images, not used for model training.
- **Scope of test separation:** training and test images were from the same barn, camera and collection period. Thus, the split measures performance on unseen images from the *same installation*, not generalization to other installations.

The model was a one-class **Mask R-CNN with a ResNet-50 backbone and Feature Pyramid Network (FPN)**, implemented with Detectron2 and PyTorch. Training was initialized from Detectron2 model-zoo weights for `mask_rcnn_R_50_FPN_3x` (COCO instance segmentation), rather than from randomly initialized parameters. The backbone of these pretrained weights originated from ImageNet-trained ResNet-50. Classification and mask layers were configured for **one foreground class** and the model was fine-tuned using the feeder annotations.

The model-zoo base configuration was:

```text
COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml
MODEL.ROI_HEADS.NUM_CLASSES = 1
```

Mask R-CNN jointly estimates object classification, a bounding box and a segmentation mask. Only the **bounding-box coordinates** were used for extraction of images to pass to the leftover-weight regression stage.

## Detector training settings

The annotated dataset was registered and fitted using Detectron2's `DefaultTrainer`. The study manuscript reports the following configuration:

| Parameter | Reported setting |
| --- | --- |
| Training images | 30 |
| Images per training iteration | 5 |
| Base learning rate | 0.00025 |
| Maximum iterations | 600 (approximately 100 passes through 30 images at batch size 5) |
| RoI samples per image | 128 |
| Number of foreground classes | 1 (`feeder`) |
| Initialization | COCO-pretrained Mask R-CNN R50-FPN (`3x`) |
| Optimizer | Detectron2 default SGD with momentum |
| Learning-rate schedule | Detectron2 default schedule, including linear warm-up |
| Training augmentation | Random horizontal flip and multiscale input resizing (Detectron2 defaults) |
| Initial state | Pretrained weights, not a resumed detector-training checkpoint |
| Training hardware | MacBook Pro, Apple M2 Pro CPU, 32 GB RAM; 6 data-loading workers |
| Reported operating system | macOS 27.0.1 |

Parameters not specified above followed the Detectron2 defaults for the selected configuration, according to the manuscript. Exact default values can be version dependent and should therefore be recovered from the environment and *resolved training configuration* when reproducing the fitting step.

The manuscript reports **Python 3.8.18, Detectron2 0.6, PyTorch 2.4.1, OpenCV 4.13.0 and NumPy 1.24.4** for feeder detection and evaluation. These are *reported research-environment versions*, not requirements for the distributed GUI.

## Evaluation of segmentation/detection on 20 held-out images

Ground-truth segmentation masks were obtained by rasterizing the LabelMe feeder polygons with OpenCV's `fillPoly`. Predicted masks were taken from Mask R-CNN output and compared with ground truth as binary arrays (using NumPy operations). For this **detector evaluation**, predicted instances with **confidence below 0.50** were discarded.

For the predicted mask \(P\) and annotated mask \(G\):

\[
\mathrm{IoU}=\frac{|P\cap G|}{|P\cup G|},\qquad
\mathrm{Dice}=\frac{2|P\cap G|}{|P|+|G|}.
\]

IoU and Dice were computed for each test screenshot and then averaged over the 20 images. Segmentation **mAP@[0.50:0.95]** was computed using mask-based matching across IoU thresholds **0.50, 0.55, ..., 0.95**, rather than from bounding-box overlap. The manuscript describes a custom Python implementation of the COCO-style matching procedure.

| Held-out detector test metric (n = 20) | Reported result |
| --- | ---: |
| Mean IoU | 0.92 |
| Mean Dice | 0.96 |
| Segmentation mAP@[0.50:0.95] | 0.91 |

These scores assess localization and segmentation of the feeder; they do **not** assess the accuracy of the subsequent leftover-weight regression.

## Timestamp matching, video sampling and crop selection

The researcher recorded timestamps for reference observations and matched them to the on-site video using experimental records and, when applicable, recording metadata. For each reference observation, the manuscript describes extraction of a **4-second video clip centered on the target timestamp (±2 s)**. Frame sampling was carried out at **0.20-second intervals**.

The published research workflow can be followed conceptually as:

1. Identify the target timestamp in the experiment record and locate the corresponding camera footage.
2. Extract the **4-second (±2 s)** interval around that timestamp.
3. Decode and sample the frames at **0.20-s intervals**.
4. Run the trained feeder detector on sampled frames.
5. Retain detections that meet the **minimum acceptance confidence of 0.90**.
6. Choose the **highest-confidence eligible detection across the clip**.
7. Extract that detection's **axis-aligned rectangular bounding-box crop**; preserve its image/frame identifier and detector score where available.
8. Convert the crop to the format required by EfficientNet V2-S, as described in [section 02](02_leftover_weight_estimation.md).

**Distinguish the detection-score thresholds:** the **0.50 threshold** above belongs to evaluation of Mask R-CNN against annotated detector-test images; **0.90** is the rule for accepting a crop in the *leftover-weight estimation* workflow. Earlier documented analysis settings also mention `SCORE_THRESH_TEST = 0.30` for retrieving candidate instances: that lower **candidate-generation** value, where used, is **not** the final crop-acceptance threshold. The runtime candidate-generation setting should be checked against the checkpoint's accompanying configuration before independently reproducing crop selection.

The selected cropped image, **not a segmentation-mask cutout**, is passed to EfficientNet. **One clip produces one selected crop and one mass prediction**; the study does not average predictions from all sampled frames. If no detection meets the acceptance threshold, it is a **failed/missing extraction**, not a measured or predicted zero-gram leftover.

## Reproducibility and limitations

Reproduction requires the same trained detector checkpoint and preprocessing/selection settings, as well as the annotated 30/20 image split and (for pixel-level metric reproduction) the evaluation script. These research inputs and scripts are **not included in this documentation-only repository**. The GUI release distributes trained checkpoints for **inference**, not the detector-training dataset or original source code. The recorded 4-s and 0.20-s settings are research-method settings; the GUI's user-selectable clip-sampling default is documented separately in [`gui/README.md`](../gui/README.md).

The detector was evaluated on one camera/feeder configuration and should be checked before deployment on other camera angles, feeder shapes, image resolutions or lighting conditions.
