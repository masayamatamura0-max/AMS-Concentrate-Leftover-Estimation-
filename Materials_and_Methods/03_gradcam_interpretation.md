# 03 — Interpretation of weight predictions using Grad-CAM

[← Research overview](../README.md) · [01 — Object detection](01_object_detection.md) · [02 — Leftover-weight estimation](02_leftover_weight_estimation.md)

## Objective and interpretation target

**Gradient-weighted Class Activation Mapping (Grad-CAM)** was used to visualize spatial image features associated with **EfficientNet V2-S's predicted concentrate leftover weight**. This is **regression Grad-CAM**, not a classification heatmap: the scalar target for differentiation was the trained model's **log-transformed regression output**, \(\widehat{y}=\widehat{\ln(1+w)}\), not a predicted class score or the final back-transformed grams.

The analyzed input was the **same rectangular feeder crop** used for mass prediction (section 02), including any visible background within that box. Although Mask R-CNN provided an instance mask during feeder detection, its mask was **not** applied to erase the background before Grad-CAM or EfficientNet regression.

## Model state and image preprocessing

Grad-CAM was computed using the **trained EfficientNet V2-S checkpoint in evaluation mode**, with **no additional model training** and **no random image augmentation**. Crops were converted to **RGB**, resized to **224 × 224 pixels** and normalized with ImageNet means **(0.485, 0.456, 0.406)** and standard deviations **(0.229, 0.224, 0.225)**. Heatmaps were drawn over the corresponding original RGB crop after resizing to the 224 × 224 display size.

The supplied manuscript identifies the target as a **late-stage convolutional feature-extraction block**. The preceding repository documentation identifies the selected PyTorch module as **`model.features[6]`**. The exact module/hook placement should be checked in the executed visualization script and saved model definition before claiming bit-for-bit reproduction of the original heatmaps.

## Computational procedure

1. Load the frozen pretrained-and-fine-tuned **EfficientNet V2-S** regression checkpoint and set it to evaluation mode.
2. Preprocess the selected feeder crop exactly as for inference (RGB, 224 × 224, ImageNet normalization).
3. Register forward/backward hooks on the selected late-stage convolutional feature block to obtain activations \(A^k_{ij}\) and their gradients with respect to the **scalar log-scale regression output** \(\widehat y\).
4. Perform a forward pass and backpropagate from \(\widehat y\) to compute \(\partial \widehat y/\partial A^k_{ij}\).
5. Spatially average the gradients within each feature channel to obtain importance weights \(\alpha_k\).
6. Calculate the weighted sum of feature maps and apply the rectified linear unit (ReLU).
7. Normalize the activation map to **0–1** for visualization, resize it to **224 × 224 pixels by bilinear interpolation**, and render it using the **JET colormap**.
8. Overlay the heatmap on the original resized crop using **45% heatmap opacity** and **55% original image**.

For a feature map of \(Z\) spatial positions in each channel \(k\), the channel weight and the unnormalized display map are:

\[
\alpha_k=\frac{1}{Z}\sum_i\sum_j\frac{\partial\widehat y}{\partial A^k_{ij}},
\]

\[
L_{\mathrm{Grad-CAM}}=\mathrm{ReLU}\left(\sum_k\alpha_k A^k\right).
\]

The resulting map is scaled to the **0–1** display interval before conversion to an overlay. The JET colors and per-image normalization are display conventions, **not calibrated grams or contribution coefficients**.

## Interpretation and limitations

Grad-CAM is a **qualitative sensitivity visualization** for the selected regression output. A prominent activation does not establish that a visible pellet caused a particular predicted value, nor does a low activation establish absence of a feature's influence. The highlighted regions may include pellets, dust, bowl edges, glare, illumination patterns and other image features within the bounding box.

Because each heatmap is normalized separately, the intensity of red or blue areas should **not** be compared as an absolute importance scale between different photographs. Grad-CAM neither quantifies prediction uncertainty nor replaces evaluation against manually weighed leftovers. Interpret heatmaps alongside the independent-test errors and the weight distribution described in [section 02](02_leftover_weight_estimation.md).

## Parameters required to reproduce the figures

| Component | Reported or documented setting |
| --- | --- |
| Trained model | EfficientNet V2-S regression checkpoint |
| Input | Rectangular feeder bounding-box crop, RGB 224 × 224 |
| Normalization | ImageNet RGB mean and SD |
| Differentiation target | Scalar **log1p-scale** regression output |
| Feature block | Late-stage convolutional block; preceding repository notes identify `model.features[6]` |
| Channel weights | Spatial mean of gradients |
| Map construction | Weighted sum of feature maps followed by ReLU |
| Display range | Individually normalized to 0–1 |
| Heatmap size | 224 × 224 via bilinear interpolation |
| Color map | JET |
| Overlay | 45% heatmap + 55% original crop |
| Model behavior | Evaluation mode; no random augmentation or training |

## Code and data availability

The repository documents the procedure, but the original **Grad-CAM script, exact checkpoint-to-feature-hook mapping and analyzed research images** are not included. Consequently, the figure-generation settings are specified for transparency, but pixel-for-pixel heatmap reproduction requires those additional artifacts.
