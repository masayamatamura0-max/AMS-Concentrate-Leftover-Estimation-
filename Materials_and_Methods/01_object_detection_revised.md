# 01 — Feeder detection and image extraction (Mask R-CNN)

[← Research overview](../README.md) · [02 — Leftover-weight estimation](02_leftover_weight_estimation.md) · [03 — Grad-CAM](03_gradcam_interpretation.md)

This page describes the feeder-detection and image-extraction methods and results from the study *Estimation of automated milking system concentrate leftovers with 2-dimensional images and machine learning* (Thomas et al.; JDS Communications manuscript). The detector identifies the feeder in images from an automated milking system (AMS) and provides a rectangular crop for the subsequent EfficientNet V2-S model.

## Materials and methods

### Camera and annotated images

A Reolink RLC-511W camera was mounted on the ceiling of the AMS room at the University of Guelph's Ontario Dairy Research Centre (Elora, Ontario, Canada), approximately 2.3 m above the feeder. Fifty PNG screenshots were obtained from video recordings at different times of day and with different amounts of leftover concentrate between November 11 and 17, 2025. The screenshots had an average resolution of approximately 1,376 × 1,032 pixels.

The feeder was manually outlined in each screenshot as a closed polygon using LabelMe. The annotations were assigned a single foreground class (`feeder`) and converted from individual LabelMe JSON files to a combined COCO-format instance-segmentation annotation file. Thirty images were used for training, and the remaining 20 were held out for testing. Both subsets came from the same camera, barn, and collection period.

### Mask R-CNN training

Feeder localization used Mask R-CNN with a ResNet-50 backbone and Feature Pyramid Network (FPN), implemented in Detectron2 and PyTorch. The model was initialized from the COCO-pretrained `mask_rcnn_R_50_FPN_3x` model, and its classification and mask-prediction layers were configured for one foreground class. The network was then fine-tuned on the annotated feeder images.

| Training parameter | Setting |
| --- | --- |
| Base configuration | `COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml` |
| Foreground classes | 1 (`feeder`) |
| Training images | 30 |
| Batch size | 5 images/iteration |
| Base learning rate | 0.00025 |
| Maximum training iterations | 600 (approximately 100 passes through the training images) |
| Region-of-interest samples per image | 128 |
| Initialization | COCO-pretrained Mask R-CNN R50-FPN (3× schedule) |
| Optimization | Stochastic gradient descent with momentum and linear learning-rate warm-up (Detectron2 configuration defaults) |
| Data augmentation | Random horizontal flipping and multiscale image resizing |
| Computing environment | MacBook Pro (Apple M2 Pro CPU, 32 GB RAM); 6 data-loading workers |

The detector was trained using Detectron2's `DefaultTrainer`. Feeder-detection analyses used Python 3.8.18, Detectron2 0.6, PyTorch 2.4.1, OpenCV 4.13.0, and NumPy 1.24.4.

### Evaluation of feeder segmentation

The trained model was evaluated on the 20 held-out screenshots. Ground-truth binary masks were generated from the manually annotated polygons using OpenCV `fillPoly`, and predicted binary masks were obtained from Mask R-CNN. Predictions with detection confidence below **0.50** were excluded from this detector evaluation.

For predicted mask *P* and ground-truth mask *G*, intersection over union (IoU) and the Dice similarity coefficient were calculated as:

\[
\mathrm{IoU} = \frac{|P \cap G|}{|P \cup G|}
\]

\[
\mathrm{Dice} = \frac{2|P \cap G|}{|P| + |G|}
\]

IoU and Dice were calculated for each image and averaged across the test set. Segmentation mean average precision (mAP@[0.50:0.95]) was evaluated by matching predicted and ground-truth **masks** at IoU thresholds from 0.50 to 0.95 in increments of 0.05, using a COCO-style evaluation procedure. This was a mask-level metric, not bounding-box average precision.

### Video-frame selection and feeder cropping

For leftover-weight estimation, the camera recordings were approximately **2,560 × 1,920 pixels at 20 frames/s**. Experimental timestamps, supplemented by recording metadata where applicable, were used to identify the footage associated with each reference weight. A **4-second clip centered on the target timestamp (±2 s)** was extracted and sampled at **0.20-second intervals**.

Mask R-CNN was applied to the sampled frames. For each clip, detections with confidence scores of at least **0.90** were eligible, and the **highest-confidence eligible detection** was selected. Its axis-aligned **rectangular bounding box** was cropped to create a single feeder image for the EfficientNet V2-S regression model.

**The regression input was the bounding-box crop, not a segmentation-mask cutout.** Although Mask R-CNN predicted feeder masks, these masks were **not** used to remove background pixels. Any feeder edges or surrounding image content inside the bounding box remained visible. The **0.90** crop-acceptance threshold is distinct from the **0.50** threshold used for evaluation of feeder segmentation.

The subsequent resizing, RGB conversion, normalization, and regression procedures are described in [02 — Leftover-weight estimation](02_leftover_weight_estimation.md).

## Results

### Feeder-detection performance

The detector achieved the following results on the **20 held-out screenshots**:

| Metric | Result |
| --- | ---: |
| Mean IoU | **0.92** |
| Mean Dice similarity coefficient | **0.96** |
| Segmentation mAP@[0.50:0.95] | **0.91** |

The total training loss decreased from approximately **1.86** at iteration 19 to **0.11** at iteration 599. Training took approximately **80 minutes**. These results indicate close agreement between detected feeder regions and manual annotations within the studied camera and feeder setup.

### Extracted feeder images

One bounding-box crop was selected per video clip for regression. The **315 model-development crops** had a median width of **426 pixels** (range: 397–466) and a median height of **445 pixels** (range: 426–467). Among the **92 independent-test images**, 90 had a median width of **413 pixels** (range: 389–469) and a median height of **433 pixels** (range: 409–467). The other two test images had lower resolutions (**188 × 197** and **194 × 203 pixels**) because their original recordings were unavailable.

The detection scores above assess **feeder localization and segmentation**, not leftover-weight prediction accuracy. Regression training and test performance are described in [02 — Leftover-weight estimation](02_leftover_weight_estimation.md).

## Interpretation and scope

The method accurately localized the feeder in held-out images acquired from the **same AMS installation**. Its performance on different AMS brands, feeder geometries, camera placements, or lighting conditions was not established by this detector test. For reproduction of the exact numerical results, the original training/test images, annotations, trained checkpoint, software configuration, and evaluation code are required.
