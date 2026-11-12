# 03 — Grad-CAM interpretation of leftover-weight predictions

[← Repository homepage](../README.md) · [01 — Object detection](01_object_detection.md) · [02 — Leftover-weight estimation](02_leftover_weight_estimation.md)

## Purpose

**Gradient-weighted Class Activation Mapping (Grad-CAM)** was used to examine which spatial regions of a cropped feed-bowl image were associated with the trained **EfficientNet V2-S** regression output. Unlike a conventional classification Grad-CAM, the target was a **single continuous prediction** of transformed leftover weight, not a class probability.

## Regression Grad-CAM procedure

The documented workflow used a late-stage EfficientNet feature-extraction block, **`model.features[6]`**, to generate the activation map.

1. Prepare each feed-bowl crop as an **RGB 224 × 224** input using the same ImageNet normalization applied during regression inference.
2. Register forward and backward hooks for the selected feature block to collect feature activations and output gradients.
3. Compute the model's scalar **log1p-scale** prediction and backpropagate that output to the selected feature block.
4. Average the gradients spatially within each channel to obtain channel weights.
5. Compute a weighted combination of the recorded activation maps and apply a rectified linear unit (ReLU).
6. Normalize the resulting two-dimensional activation map for display, resize it to the crop dimensions, and overlay it on the original crop as a heatmap.

For feature-map channel \(k\), spatial position \((i,j)\), and scalar model output \(y\), the channel weight is:

$$
\alpha_k = \frac{1}{Z}\sum_i\sum_j \frac{\partial y}{\partial A_{ij}^{k}},
$$

where \(A_{ij}^{k}\) is the activation at that location and \(Z\) is the number of spatial positions in the feature map. The display map is:

$$
L_{\mathrm{GradCAM}} = \operatorname{ReLU}\!\left(\sum_k \alpha_k A^k\right).
$$

Each heatmap was normalized to a **0–1** display range before it was overlaid on the corresponding image. Overlay color and transparency are visualization choices, not measurements of residual mass.

## Interpretation and limitations

Grad-CAM provides a **qualitative indication of image regions associated with a prediction**. It does **not** identify causal contributions of individual feed particles, establish that the network learned only feed-relevant features, or quantify uncertainty in the predicted weight.

Activation patterns can be affected by crop framing, visible bowl edges, background, reflections, and lighting. Heatmaps normalized separately for each image must not be interpreted as directly comparable absolute importance values across observations. Grad-CAM should therefore be considered alongside independent prediction-error analysis, rather than as a substitute for it.

## Reporting and availability

When presenting Grad-CAM figures, document the trained checkpoint, preprocessing, feature block, scalar target, display normalization, and overlay settings. The written procedure is provided to support interpretation of the study; the underlying Grad-CAM script and analysis notebooks are not publicly distributed here.
