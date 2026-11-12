# 03 — Interpretation of leftover-weight predictions using Grad-CAM

[← Research overview](../README.md) · [01 — Object detection](01_object_detection.md) · [02 — Leftover-weight estimation](02_leftover_weight_estimation.md)

## Objective

Gradient-weighted Class Activation Mapping (Grad-CAM) was used to visualize image regions associated with predictions from the trained EfficientNet V2-S **regression** model described in [section 02](02_leftover_weight_estimation.md). Unlike classification Grad-CAM, the differentiation target was the model's **scalar prediction on the log-transformed weight scale**, $\hat{y}$, rather than a class score or a prediction back-transformed to grams.

## Grad-CAM calculation

Grad-CAM was generated with the trained model in evaluation mode, without additional training or random image augmentation. The same feeder crops and preprocessing used for weight prediction were used for visualization (see [section 02](02_leftover_weight_estimation.md)). Activations and gradients were obtained from a late-stage convolutional feature-extraction block.

Let $A^k_{ij}$ denote the activation at spatial position $(i,j)$ in feature channel $k$, and let $Z$ be the number of spatial positions in that channel. Each channel was assigned a weight equal to the spatial mean of the gradient of the regression output:

$$
\alpha_k = \frac{1}{Z}\sum_i\sum_j\frac{\partial\hat{y}}{\partial A^k_{ij}}
$$

The Grad-CAM activation map was then calculated as:

$$
L_{\mathrm{Grad\text{-}CAM}} = \mathrm{ReLU}\left(\sum_k\alpha_k A^k\right)
$$

The resulting activation map was normalized to a range of **0–1**, resized to **224 × 224 pixels** using bilinear interpolation, and displayed with the **JET colormap**. Heatmaps were superimposed on the corresponding feeder images using **45% heatmap** and **55% image** blending.

## Interpretation and limitations

Grad-CAM provides a **qualitative visualization** of image regions associated with the predicted leftover weight. Highlighted regions may include concentrate pellets, dust, feeder edges, or lighting features; they do not establish causal contributions to the prediction. Because each heatmap was normalized independently, color intensity should not be interpreted as an absolute measure of importance or compared quantitatively between images. The heatmaps do not measure prediction error or uncertainty; predictive accuracy was evaluated separately against reference weights, as described in [section 02](02_leftover_weight_estimation.md).

## Reproducibility note

The earlier repository notes identify `model.features[6]` as the Grad-CAM target layer, whereas the manuscript describes it more generally as a late-stage convolutional feature-extraction block. The exact hook location should be verified against the executed Grad-CAM script and trained model architecture before attempting pixel-level reproduction of the heatmaps.
