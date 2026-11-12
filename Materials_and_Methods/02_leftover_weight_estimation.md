# 02 — Estimation of concentrate leftover weight

[← Repository homepage](../README.md) · [01 — Object detection](01_object_detection.md) · [03 — Grad-CAM](03_gradcam_interpretation.md)

## Objective and reference dataset

The regression task was to estimate the **mass (g) of concentrate remaining in the AMS feed bowl** from the rectangular bowl image selected by the object-detection stage. Known-weight images were used for model development, with separately collected naturally occurring leftovers reserved for independent evaluation.

The documented controlled-weight development set comprised **315 cropped images** with reference weights from **0 to 1,000 g in 50-g increments** (21 levels, 15 images per level). The final model-development partition used **252 training images** and **63 internal-validation images** (80:20), stratified by exact reference weight, with a recorded random seed of **42**.

A separate collection initially comprised **92 naturally occurring leftover observations**. **Two observations were excluded**, leaving **90 observations for the final independent evaluation**. Exclusion criteria should be reported consistently with the final manuscript and observation manifest; this repository does not assign a reason not established in the documented record. The independent observations were not used to fit model weights, choose a checkpoint, or select training hyperparameters.

The controlled reference-weight interval is not a guarantee of accuracy beyond 1,000 g. Moreover, a separate evaluation set acquired in the same general recording environment does not establish transferability to unfamiliar farms, feed bowls, or cameras.

## Image preparation

Each input crop was converted to **RGB**, resized to **224 × 224 pixels**, converted to a tensor, and normalized using the ImageNet channel statistics:

| Channel | Mean | Standard deviation |
| --- | ---: | ---: |
| Red | 0.485 | 0.229 |
| Green | 0.456 | 0.224 |
| Blue | 0.406 | 0.225 |

During training, random image augmentation was applied, including **horizontal flipping** (probability 0.5) and photometric adjustments to brightness, contrast, saturation, and hue. Documented jitter magnitudes were **0.2** for brightness/contrast/saturation and **0.03** for hue. Internal-validation and independent-evaluation crops were processed deterministically, without random augmentation.

## EfficientNet V2-S architecture

**EfficientNet V2-S**, implemented using `torchvision.models.efficientnet_v2_s`, was initialized from ImageNet-1K pretrained weights during training. The final classification layer was replaced with a **single-output linear layer**, allowing the network to predict a continuous response instead of a classification probability. The resulting model produced one scalar for each input crop.

## Response transformation, fitting, and checkpoint selection

To fit the regression network, reference weight (in grams) was transformed with the natural logarithm:

$$
y = \ln(1+w), \qquad w = \text{reference leftover weight (g)}.
$$

The training loss was **mean squared error (MSE) on the transformed response scale**. The methodological record lists the following settings:

| Setting | Documented value |
| --- | --- |
| Optimizer | AdamW |
| Initial learning rate | `5 × 10⁻⁵` |
| Weight decay | `1 × 10⁻⁴` |
| Batch size | 16 |
| Maximum epochs | 200 |
| Early-stopping patience | 20 epochs |
| Minimum validation-loss improvement | 0.0001 |
| Checkpoint selection | Lowest internal-validation MSE |

The independent evaluation observations were kept separate from optimization, early stopping, and model selection to avoid test-set leakage.

## Weight prediction in grams

The network's output is a prediction on the `log1p` scale. It is converted back to grams by:

$$
\widehat{w}_{\mathrm{g}} = \max\!\left(0,\exp(\widehat{y})-1\right).
$$

Negative predictions after inverse transformation are clipped to 0 g. This numerical constraint does not mean that failed object detections are assigned zero mass.

For a video clip, **one selected feed-bowl crop** is supplied to EfficientNet, producing **one prediction**. Predictions are not averaged over all sampled video frames.

## Model evaluation

Predictive performance was assessed on weights expressed in **grams** after inverse transformation. Let \(w_i\) be the reference weight and \(\widehat{w}_i\) the predicted weight for the \(i\)-th observation, with \(n\) observations. The evaluation metrics are:

$$
\mathrm{RMSE}=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(w_i-\widehat{w}_i)^2},
$$

$$
\mathrm{MAE}=\frac{1}{n}\sum_{i=1}^{n}|w_i-\widehat{w}_i|,
$$

$$
R^2=1-\frac{\sum_{i=1}^{n}(w_i-\widehat{w}_i)^2}{\sum_{i=1}^{n}(w_i-\overline{w})^2}.
$$

Internal-validation results support checkpoint selection; performance on the **90 independent naturally occurring observations** assesses application beyond controlled known-weight images. Accuracy summaries should report the evaluation sample size, distribution of reference masses, and any separately analyzed range restrictions or exclusions. In particular, observed values above the **0–1,000 g development range** should be identified as outside that range.

Numerical accuracy estimates are intentionally not reproduced here without the finalized results table and matched observation manifest. Performance statistics and their treatment of exclusions must be consistent with the accepted manuscript.

## Model availability

A trained `EfficientNet.pth` checkpoint is included in the [Windows GUI release](../../../releases/latest) for inference. Python training and evaluation code, Jupyter notebooks, and the complete research dataset are not published in this repository.
