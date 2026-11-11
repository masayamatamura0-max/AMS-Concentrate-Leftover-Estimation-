# Materials and Methods — Object detection and image extraction

[Repository homepage](../README.md) · [Study design](00_study_design_and_data.md) · [Leftover-weight estimation](02_leftover_weight_estimation.md)

## Aim

The object-detection stage finds the concentrate feed bowl in a full-resolution camera frame and defines an image region for estimating leftover mass. This stage does **not** predict weight; the subsequent EfficientNet model does that.

## Detection architecture

The documented detector uses **Mask R-CNN** implemented in **Detectron2**, configured as a **one-class** detector of the AMS feed bowl. The base configuration is reported as:

```text
COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml
MODEL.ROI_HEADS.NUM_CLASSES = 1
```

The checkpoint was trained for the target feeder. The original annotation dataset, detector-training hyperparameters, and final stand-alone detector accuracy are **not included** in this repository; do not infer or invent those details from the inference code. A custom checkpoint needs the corresponding network configuration.

### Candidate generation versus acceptance

The documented research notebook can use an internal Detectron2 test-score threshold as low as `SCORE_THRESH_TEST = 0.30` to retrieve candidates. That **candidate-generation threshold** is separate from the final criterion used to accept the best bowl crop. The reported **minimum accepted detection confidence is 0.90**.

For a frame with multiple predictions, consider only candidate instances permitted by the chosen threshold, then rank by their detector confidence score. These scores reflect the detector's classification confidence and are **not calibrated uncertainty estimates for feed mass**.

## Sampling and selection from a clip

The research workflow decodes short clips using OpenCV and examines selected frames, optionally using transformed orientations in the research notebook. The general best-crop procedure is:

1. Sample frames at the research-run frame interval.
2. Apply the trained Mask R-CNN detector to each considered frame/orientation.
3. Identify the candidate feed-bowl instance with the **highest confidence score** across all considered frames.
4. Accept the crop if the selected candidate meets the minimum confidence criterion (**0.90** in the documented configuration).
5. Use its **bounding-box coordinates** to crop the corresponding original (or considered-orientation) frame, and preserve metadata about the selected frame and score.

Although Mask R-CNN predicts instance-segmentation masks, **the input for EfficientNet is a rectangular bounding-box crop**. The mask is **not** applied to remove all pixels outside the bowl. This distinction matters because visible surroundings may remain in the crop.

## Important distinction: archived notebook versus current GUI

The archived October research notebook does not identify one unambiguous sampling configuration across all cells and helper functions:

| Implementation detail | Archived research notebook | Current Windows GUI |
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

## Related files

When the GUI source is added to the repository, `gui/detection.py` will contain detector inference and `gui/app.py` will contain frame selection, overlay drawing, and cropping. Those **application files are supplied separately** from this documentation package.
