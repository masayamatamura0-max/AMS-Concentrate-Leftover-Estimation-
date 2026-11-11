# Materials and Methods — Model evaluation and reporting

[Repository homepage](../README.md) · [Leftover-weight estimation](02_leftover_weight_estimation.md) · [Grad-CAM](04_gradcam_interpretation.md)

## Evaluation sets

The reported development dataset was separated into **training (n = 252)** and **internal validation (n = 63)**. Internal validation was used to select the EfficientNet checkpoint and stop training. The naturally occurring dataset initially comprised **92 observations**; **two were excluded**, leaving the **final independent evaluation set at n = 90**. The exclusion criteria should be reported explicitly in the manuscript and corresponding data manifest. The test observations were not intended for model selection.

Before treating the reported partitions as fully independent, confirm whether multiple images or clips derive from the same physical observation, cow, or session. If observation-level dependence is present, document it and consider grouped validation as an additional robustness analysis. **Independent observations collected in the same AMS environment do not imply external validation across farms.**

## Prediction-scale conventions

Predictions are returned to grams from the model's `log1p` response using `max(0, expm1(output))`. Calculate reported prediction errors using **reference masses and predictions in grams**, rather than accidentally computing metrics on transformed targets.

For observation $i$, let $y_i$ be its reference mass (g), $\hat y_i$ the predicted mass (g), and $n$ the number of test observations:

**Root mean squared error**

$$
 \mathrm{RMSE}\ (\mathrm{g})=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2}.
$$

**Mean absolute error**

$$
 \mathrm{MAE}\ (\mathrm{g})=\frac{1}{n}\sum_{i=1}^{n}|y_i-\hat y_i|.
$$

**Coefficient of determination**

$$
 R^2=1-\frac{\sum_i(y_i-\hat y_i)^2}{\sum_i(y_i-\bar y)^2}.
$$

This definition of R² can be **negative** when predictions perform worse than the reference-mean benchmark. Do not silently replace negative values with zero.

## Recommended descriptive summaries and figures

For each evaluated set, report: sample count, reference-mass mean and standard deviation, median, minimum, maximum, number/proportion exceeding the model-development range, and number of invalid or missed detections. A reference-versus-predicted scatter plot should include a **1:1 line**. Residuals or absolute errors versus reference weight can reveal variation in accuracy across the full weight range.

The development range is **0–1,000 g**. An earlier project summary described reference masses reaching approximately **1,459 g** in the original observations; the maximum and distribution of the **final 90-observation analysis set** must be recalculated from its manifest. Weight estimates beyond the development range require explicit caution, even if the detector has high confidence.

## Outlier and high-weight sensitivity analyses

The archived analysis documentation identifies **different high-weight filtering rules** in different research functions: for example, one excluded observations with reference weight **≥ 800 g**, and another excluded **≥ 1,000 g**. **Do not infer that either historical threshold explains the two exclusions from the final 90-observation analysis.** Confirm the actual selection rule and excluded observations from the final data manifest.

A transparent reporting approach is:

1. Describe the **92 initial observations** and the **2 documented exclusions**, then report the final **n = 90** evaluation results.
2. State any sensitivity rule **before interpreting the corresponding results**, including the cutoff and whether the comparison is inclusive or exclusive.
3. Report the number of observations removed and their observed reference weights, when permitted.
4. Report each sensitivity result separately with its resulting sample size.
5. Explain that trimming the difficult tail does not make the full-range performance equal to the restricted-range performance.

These steps improve transparency and avoid changing thresholds post hoc to maximize apparent accuracy.

## Object-detection success versus weight-prediction accuracy

The two stages of the pipeline have different failure modes. Detection confidence is not the same as measurement uncertainty or regression quality. If no bowl is detected, the output should be recorded as **missing/failed**, not 0 g. Document the proportion of observations for which a valid crop is obtained, because reporting regression errors only on successfully processed observations can hide failures of the overall pipeline.

The GUI defaults to **1.00-s clip sampling** without 180° rotations, whereas the research notebook describes other candidate-selection settings. GUI speed improvements do not prove equivalent model accuracy; compare them on the same held-out observations before claiming equivalence.

## What this repository can and cannot establish

The documentation summarizes the described evaluation design, but it contains **no raw reference weights, predicted masses, original result tables, or final executed evaluation notebook**. Consequently, no RMSE, R², MAE, or statistical significance values are invented in this repository. Add verified result tables only after reconciling manuscript values and evaluation code.

## Reporting checklist

- [ ] Reference dataset and independent-test sample sizes confirmed.
- [ ] Grouping and potential repeated-measurement leakage reviewed.
- [ ] Test data excluded from tuning and model selection.
- [ ] Regression metrics calculated after back-transformation to grams.
- [ ] Missed detections and unreadable files counted separately.
- [ ] Full-sample evaluation reported before any restricted-range sensitivity analysis.
- [ ] High-weight exclusion rules and resulting sample sizes explicitly documented.
- [ ] Model-development range and independent-test range reported.
- [ ] Limitations for unfamiliar farms, camera settings, and feeder geometries discussed.
