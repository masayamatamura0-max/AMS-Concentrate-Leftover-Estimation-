# Materials and Methods — Object detection and image extraction

[Repository homepage](../README.md) · [Leftover-weight estimation](02_leftover_weight_estimation.md)

## Aim

The object-detection stage finds the concentrate feed bowl in a full-resolution camera frame and defines an image region for estimating leftover mass. This stage does **not** predict weight; the subsequent EfficientNet model does that.

## Detection architecture

The documented detector uses **Mask R-CNN** implemented in **Detectron2**, configured as a **one-class** detector of the AMS feed bowl. The base configuration is reported as:

```text
COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml
MODEL.ROI_HEADS.NUM_CLASSES = 1
```

### Candidate Generation and Detection Acceptance

During object detection, `SCORE_THRESH_TEST = 0.30` was used to generate candidate detections. This threshold was distinct from the minimum confidence score of 0.90 required to accept a detected feed bowl for subsequent image cropping. When multiple candidate detections were identified within a frame, the detection with the highest confidence score was selected, provided that its score met the acceptance threshold of 0.90. These confidence scores represent the object detector's classification confidence rather than uncertainty estimates associated with leftover-weight prediction.

## Sampling and selection from a clip

The research workflow decodes short clips using OpenCV and examines selected frames. The general best-crop procedure is:

1. Sample frames at the research-run frame interval.
2. Apply the trained Mask R-CNN detector to each considered frame/orientation.
3. Identify the candidate feed-bowl instance with the **highest confidence score** across all considered frames.
4. Accept the crop if the selected candidate meets the minimum confidence criterion (**0.90** in the documented configuration).
5. Use its **bounding-box coordinates** to crop the frame, and preserve metadata about the selected frame and score.
6. 
## Research processing versus current GUI

The original internal research records describe more than one sampling configuration. Because the executed research files are not provided here, the exact settings used for the reported scientific evaluation need confirmation:

| Implementation detail | Internal research records | Current Windows GUI |
| --- | --- | --- |
| Sampling interval | `0.05 s` in one constant; `0.20 s` in helper defaults/diagnostics; confirm executed call | **1.00 s** by default, adjustable |
| Artificial 180° frame rotation | May be enabled in the research workflow | **Not performed** |
| Detectron2 passes per sampled frame | Original and possibly rotated candidate | **One original-frame pass** |
| Crop selection | Highest accepted detector confidence | Highest accepted detector confidence |
| Image region passed to EfficientNet | Bounding-box crop | Bounding-box crop |
| EfficientNet predictions per clip | One selected crop in the described procedure | **One** selected crop |

For example, a 4-second clip at 20 frames/s contains approximately 80 frames. With a 1.00-second GUI interval, sampled frame positions are approximately **0, 1, 2, and 3 seconds**. Actual extraction depends on available video frames and valid FPS metadata.

**Do not report 1.00-second/no-rotation processing as the validated study protocol unless that protocol was independently evaluated.** The GUI settings are designed to reduce the number of detector calls.

## Full-size still images and ready-made crops

- **Full image mode:** run Detectron2 once on the still image, crop the selected feed bowl, and submit the crop to EfficientNet.
- **Crop or Clip mode (still image):** treat the image as an already prepared bowl crop; skip Detectron2.
- **Crop or Clip mode (video):** sample original frames, run Detectron2 once on each sampled frame, retain the highest-confidence crop, then call EfficientNet once.

## Reliability and reporting

Low detector confidence, visual obstruction, unusual light, an incomplete bowl, a new feeder geometry, or a corrupted video may prevent a usable crop. Absence of a detection must not be silently converted to a prediction of **0 g**. Report such cases as unprocessed or missing predictions, with counts and exclusion criteria.

A methods report should include the detector architecture, training annotation source (once verified), test configuration, candidate and acceptance thresholds, exact frame-sampling interval, rotation setting, bounding-box convention, and quality-control procedure.

## Availability of the implementation

This repository provides a written description of the detection method and a downloadable compiled GUI. **Detector-training scripts, Python inference source files, and research notebooks are not hosted here.** The trained detector checkpoint is provided in the Windows release for inference.
