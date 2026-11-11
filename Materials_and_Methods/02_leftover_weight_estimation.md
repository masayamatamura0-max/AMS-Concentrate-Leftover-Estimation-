# Materials and Methods — Estimation of concentrate leftover weight

[Repository homepage](../README.md) · [Object detection](01_object_detection.md) · [Model evaluation](03_model_evaluation.md)

## Objective and input

A regression model predicts the **mass of concentrate remaining in an AMS feed bowl, in grams**, from a cropped RGB image. The input may be (1) a bounding-box crop produced by Detectron2, or (2) an already cropped image provided directly to the GUI.

The primary model-development dataset described in the project documentation includes **315 controlled-weight images**: 15 images at each 50-g reference-weight level from **0 through 1,000 g**. Of these, **252** were allocated to training and **63** to internal validation, with a stratified 80:20 split by exact reference weight. A separate set of **92 naturally occurring** leftover observations was initially available; **two were excluded**, leaving **90 in the final independent evaluation**. See [Study design](00_study_design_and_data.md) for qualifications about grouping and data provenance.

## Image preprocessing

Each model input is processed as follows:

1. Convert the image to **RGB**.
2. Resize to **224 × 224 pixels**.
3. Convert pixel values to a tensor.
4. Normalize using ImageNet channel means and standard deviations:

| RGB channel | Mean | Standard deviation |
| --- | ---: | ---: |
| Red | 0.485 | 0.229 |
| Green | 0.456 | 0.224 |
| Blue | 0.406 | 0.225 |

This preprocessing is used for validation and inference without random augmentation.

### Training-only augmentation

The internal model-development records describe:

- Random horizontal flip, probability **0.5**;
- Small random rotation, approximately **±10°**;
- Brightness, contrast, and saturation jitter of **0.2** each;
- Hue jitter of **0.03**.

The final exact transform order should be confirmed from the finalized analysis records. **Training-only random rotations do not mean that the present GUI rotates video frames.** The current application performs no artificial 180° rotation during detection.

## CNN architecture

The documented architecture is **EfficientNet V2-S** using `torchvision.models.efficientnet_v2_s`, initialized from **ImageNet-1K pretrained weights** during model development. Its terminal classifier is replaced with a **single-output linear layer** for regression:

**Architecture summary:** EfficientNet V2-S with its final classification head replaced by a **single-output regression layer**. This is a methodological description; training code is not provided.

The inference application reconstructs the matching architecture and loads the authorized saved checkpoint without needing to download pretrained weights. A checkpoint produced with another output head or network variant may not load.

## Response transformation and training

Reference mass was transformed to stabilize regression training:

$$
 y_{\mathrm{train}}=\ln(1+w), \qquad w=\text{reference leftover mass (g)}.
$$

The model was trained with **mean squared error on this log-transformed target**, not MSE on grams.

| Hyperparameter | Value described in internal study records |
| --- | --- |
| Optimizer | AdamW |
| Learning rate | **5 × 10⁻⁵** |
| Weight decay | **1 × 10⁻⁴** |
| Batch size | **16** |
| Maximum epochs | **200** |
| Early-stopping patience | **20 epochs** |
| Minimum improvement in validation loss | **0.0001** |
| Model checkpoint selection | Best internal-validation loss |

Independent test observations must not be used for optimizer tuning, data augmentation decisions, early stopping, or model selection. Dataset counts and the model-development specification should be reconciled with the final manuscript.

## Back-transformation and inference

The model predicts a scalar on the `log1p` scale. The application converts that prediction to grams:

$$
 \widehat{w}_{\mathrm{g}} = \max\left(0,\exp(\widehat{y})-1\right).
$$

Thus a negative value after back-transformation is clipped to **0 g**. This numerical clipping must not be confused with a failed or missing object detection.

For a clip, the application does **not** predict on every sampled frame and average the results. Detectron2 selects the **highest-confidence bowl crop**, and EfficientNet predicts on **that single crop**. This choice improves efficiency but does not guarantee that the selected detector crop minimizes weight-prediction error.

## Validation and potential generalization limits

Model development is based on known reference masses within **0–1,000 g**. Predictions above that range are **extrapolations in the target variable** and should be assessed separately. An independent set from the same recording environment is not equivalent to testing on an unfamiliar farm, feeder, camera, or lighting condition.

Report errors using back-transformed predictions in **grams**; see [Model evaluation](03_model_evaluation.md). No numerical accuracy claims are made here because the current documentation package does not contain the original measured results table or raw evaluation data.

## Implementation and reproducibility boundary

The compiled Windows GUI and both trained `.pth` checkpoints are provided through [GitHub Releases](../../../releases/latest) for **inference**. **Python source files, model-training scripts, Jupyter notebooks, and the full original training dataset are not posted in this GitHub repository.** The descriptions above summarize the documented method but are not a complete executable training workflow.
