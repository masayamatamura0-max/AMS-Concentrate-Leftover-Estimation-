# Data and code availability

[Back to repository homepage](README.md)

## Public contents

This repository documents the study *Estimation of automated milking system concentrate leftovers with 2-dimensional images and machine learning* and describes the inference GUI. Its Markdown pages cover the study design, bowl detection, leftover-weight regression, evaluation, Grad-CAM, and GUI use.

Once the GUI source is added separately, the `gui/` directory provides the inference application. **A working inference GUI is not the same as a full model-training reproduction package.** The original complete research notebook, all intermediate scripts, and data manifests are not included in this documentation-only package.

## Not currently distributed

The package does **not** contain:

- Original AMS video recordings, raw camera images, or animal-level observations;
- The complete training and independent-test image datasets or private manifests;
- Feed-bowl annotation images/labels used to train the object detector;
- Trained `Detectron2.pth` and `EfficientNet.pth` weights;
- The full executed detector/CNN training and evaluation notebooks;
- Final model evaluation predictions or manuscript-ready result tables;
- A Windows binary/executable or compiled machine-learning libraries.

The project team should determine whether and how any of these may be released, considering research agreements, collaborator permissions, and privacy.

## Reproducibility scope

Methods listed in these documents were transcribed and organized from existing project records. Some details need reconciliation with the **actual executed October research notebook and final dataset manifest**, particularly:

1. Time-alignment and whether each stored clip lasted exactly 4 seconds;
2. Detector sampling interval (`0.05` versus `0.20` seconds) and use of rotated candidate frames;
3. Grouping of related images in the train/validation split;
4. The inclusion/exclusion rules for high-weight independent-test observations;
5. Exact final trained checkpoints and selected configurations.

The released GUI uses a **1.00-second/no-rotation** inference default for convenience and speed. This **must not be conflated with scientific replication** of notebook results.

## Responsible sharing

Do not commit private animal identifiers, timestamped facility video files, proprietary detector checkpoints, or data with restricted usage. For a public release, use a permitted de-identified/synthetic example and document any dataset license or access procedure approved by the project team.

## Licenses and permissions

No software or dataset license is asserted in this documentation package. A license should be chosen only with appropriate agreement from the copyright holders and research collaborators. In the absence of an explicit license, posting code or documentation publicly does not automatically grant reuse permissions.
