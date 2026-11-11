# Materials and Methods — Grad-CAM interpretation

[Repository homepage](../README.md) · [Leftover-weight estimation](02_leftover_weight_estimation.md) · [Evaluation](03_model_evaluation.md)

## Purpose

Gradient-weighted Class Activation Mapping (**Grad-CAM**) was used in the archived study workflow to visualize spatial regions associated with **EfficientNet V2-S leftover-weight predictions**. The technique supports qualitative interpretation of the trained network, but it is not a calibrated estimate of which individual feed particles causally determine the mass estimate.

## Conceptual procedure

The documented October notebook adapts Grad-CAM to a **single-output regression network**, rather than a classification logit. In outline:

1. Preprocess a feed-bowl crop as **RGB, 224 × 224**, using the same ImageNet normalization as the regression model.
2. Register forward and backward hooks on a late EfficientNet feature-extraction block; the documented notebook references **`model.features[6]`**.
3. Run a forward pass and obtain the scalar **log1p-scale** prediction.
4. Backpropagate the scalar output to obtain gradients of the selected block's activations.
5. Spatially average gradients within channels; multiply by feature activations and sum across channels.
6. Apply a rectifying operation (ReLU), resize the activation map to the crop size, and overlay it on the original image for qualitative inspection.

## Interpretation and limitations

- Heatmaps indicate image locations associated with local sensitivity of the trained prediction, **not causal effects**.
- Grad-CAM from a particular layer may emphasize broad regions rather than exact feed-particle boundaries.
- Maps normalized independently to a `[0, 1]` display range are **not quantitatively comparable across images** unless a more rigorous common scaling procedure is defined.
- The method does not validate that the detected crop or mass estimate is accurate. It should complement, not replace, held-out performance evaluation.
- Visual patterns may depend on background, bowl texture, reflections, lighting, and crop framing; these can be confounded with the amount of feed.

## Reproducibility notes

Record the exact checkpoint, preprocessing, feature layer, target scalar (before or after back-transformation), normalization, colormap, and overlay opacity when producing published figures. The complete original Grad-CAM notebook and final visualization assets are **not included** in this documentation package. Do not imply this README provides a ready-to-run Grad-CAM implementation.

## Related methods

- [Object detection](01_object_detection.md): how the feed bowl is located and cropped.
- [Leftover-weight estimation](02_leftover_weight_estimation.md): the regression model whose feature maps are visualized.
