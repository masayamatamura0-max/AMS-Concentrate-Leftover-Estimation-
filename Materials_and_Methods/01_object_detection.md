# 01 — Object detection and feed-bowl image extraction

[← Repository homepage](../README.md) · [02 — Leftover-weight estimation](02_leftover_weight_estimation.md) · [03 — Grad-CAM](03_gradcam_interpretation.md)

## Purpose and image acquisition

Automated localization of the AMS feed bowl allows an image of a consistent region of interest to be passed to the concentrate leftover-weight regression model. Detection and weight estimation are separate tasks: **Mask R-CNN identifies the bowl**, whereas **EfficientNet predicts the mass of concentrate left inside the bowl**.

The camera used in the documented workflow was positioned above the feeding area of the AMS. On-site video was recorded at approximately **2,560 × 1,920 pixels and 20 frames/s**. Short video clips were associated with reference observations using recorded timestamps. The time-alignment procedure used experiment records and, when relevant, recording metadata to find the corresponding on-site camera footage.

## Mask R-CNN detector

A **one-class Mask R-CNN** model implemented in **Detectron2** was used to locate the feed bowl. The documented configuration was based on:

```text
COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml
MODEL.ROI_HEADS.NUM_CLASSES = 1
```

This configuration uses a ResNet-50 backbone with a feature pyramid network (FPN) and a region-of-interest head trained for the target feed-bowl class. The final model used a trained detector checkpoint.

### Candidate detections and acceptance criterion

The documented Detectron2 configuration used a low **candidate-generation score threshold of 0.30** (`SCORE_THRESH_TEST`) to obtain candidate instances. The **minimum confidence score for accepting a feed-bowl detection was 0.90**. These are distinct decisions: candidate generation allows possible detections to be assessed, whereas the acceptance rule determines whether a detected region is suitable for subsequent cropping. When several candidate instances or frames were available, the pipeline retained the eligible candidate with the **highest detector confidence**.

## Selecting a crop from images or clips

For still images, Mask R-CNN predicts bowl instances, and an accepted detection is used to extract the corresponding region. For a short video, sampled frames are analyzed, and the accepted detection with the highest confidence across the considered frames is selected.

The documented pipeline follows these steps:

1. Read the still image or decode the short video using OpenCV.
2. For clips, sample frames at the selected analysis interval.
3. Run the detector and obtain candidate bowl instances and their scores.
4. Select the highest-confidence candidate meeting the **0.90** acceptance criterion.
5. Use its **axis-aligned bounding box** to extract a rectangular feed-bowl crop.
6. Pass the selected crop to the regression model, retaining the selected frame and confidence as processing metadata.

The image supplied to EfficientNet is the **bounding-box crop**, not a mask-only image with the surrounding pixels removed. Although Mask R-CNN also estimates segmentation masks, those masks are not applied as foreground-only cutouts in this regression workflow. For crop selection from 4-second video clips, frames were sampled at 0.20-second intervals.

## Quality control and scope

Detection success depends on feed-bowl visibility, image framing, illumination, and the camera/feeder configuration. If no instance meets the acceptance criterion, **no valid bowl crop is produced**.

A detector score describes confidence in object identification, **not** uncertainty in the subsequent mass estimate. Using the detector on a different camera installation or feeder geometry would require assessment of detection reliability before using its crops in the weight model.

## Method availability

The trained detector checkpoint is provided within the downloadable [Windows GUI release](../../../releases/latest) for inference. This repository supplies the written method, not the detector-training code or annotated detector-training dataset.
