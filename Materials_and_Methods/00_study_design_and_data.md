# Materials and Methods — Study design and image acquisition

[Repository homepage](../README.md) · [Object detection](01_object_detection.md) · [Leftover-weight estimation](02_leftover_weight_estimation.md)

## Objective and study setting

The objective was to estimate the weight of concentrate feed left in an automated milking system (AMS) feed bowl using two-dimensional camera images. AMS concentrate delivery represents dispensed feed; leftover weight is needed to distinguish delivery from actual consumption.

An on-site camera was mounted above the feed bowl to obtain a view suitable for automated bowl detection and subsequent weight regression. The existing project documentation describes video of approximately **2,560 × 1,920 pixels at 20 frames/s**. Lighting, mounting geometry, and camera settings are relevant to inference performance; do not assume the same accuracy will transfer to a different camera or feeder design.

## Reference masses and development images

The documented primary development set comprises images of **known concentrate leftovers** at the following reference masses:

- **0 through 1,000 g**, at **50-g intervals** (21 mass levels).
- **15 images per level**, for **315 images** in total (21 × 15).
- The source documentation mentions additional preliminary mass levels (e.g., 20, 25, and 75 g), but they were not included in the stated balanced final development set.

These are documented counts, not figures independently recalculated from raw images in this repository. Confirm them against the final dataset manifest before manuscript submission.

### Development and internal validation

The internal model-development records describe an 80:20 split:

| Partition | Images | Function |
| --- | ---: | --- |
| Training | **252** | Optimize network parameters |
| Internal validation | **63** | Early stopping, model selection, and tuning based only on development data |
| Total development images | **315** | Controlled reference weights |

The split used `train_test_split(test_size=0.2, random_state=42, stratify=true_weight)`, i.e., **stratified by exact reference mass** rather than broad weight categories. If different frames were extracted from the same physical feed sample, splitting by individual image can mix near-duplicate observations across partitions. Such dependence must be assessed using identifiers for cows, measurement events, or clips before describing validation as independent at the observation level.

### Independent evaluation

The naturally occurring dataset initially contained **92 observations**, collected separately from the controlled-weight images. **Two observations were excluded**, leaving **90 for the final independent evaluation**. The precise reasons for exclusion must be documented from the final data manifest or manuscript. An earlier project summary reported reference weights of approximately **1 to 1,459 g** (median **14.5 g**) for the original observations; these descriptive statistics **must be recalculated for the final 90-observation set** rather than assumed to remain unchanged.

The naturally occurring test observations are a useful independent observation set, but use of the same or similar on-site feeder/camera system does **not** establish generalization to new farms or imaging systems. In addition, the test set may include reference weights **above the 1,000-g development range**, which should be explicitly characterized in performance reporting.

## Video timing and short-clip preparation

The internal study records associate on-site camera recordings with observation timestamps. Two timestamp alignment routes are documented:

| Timestamp used to find the event | Seconds before event | Seconds after event | Intended segment |
| --- | ---: | ---: | --- |
| Experimental spreadsheet timestamp | 2 | 2 | 4 s centered on the event |
| iPhone metadata used to align with on-site footage | 4 | 0 | 4 s immediately preceding the event |

**Important distinction:** iPhone metadata may help locate the correct segment of the **on-site video**. This is not evidence that iPhone camera frames were part of the final on-site imaging dataset.

Some historical records refer to 5-second clips, whereas other recorded extraction settings define 4-second windows. Existing clips might have been reused without regeneration. Therefore, the final executed extraction calls, resulting video durations, timestamp time zones, and source provenance require verification before documenting one uniform video-extraction protocol.

The intended workflow is:

1. Associate each observation with a reference leftover weight and source video.
2. Establish the observation timestamp in the recording's time base (including local-time conversion where applicable).
3. Extract the short on-site video interval defined by the selected timestamp source.
4. Select a feed-bowl image automatically using the detector procedure in [Object detection](01_object_detection.md).
5. Store the resulting crop with its reference weight and observation identity.

## Data-management considerations

A reproducible study manifest should include (where permissible) a de-identified observation ID, source camera/video ID, timestamp and time zone, extraction rule, clip duration, selected frame time, detector checkpoint/configuration, detector confidence, original and crop dimensions, reference leftover mass, and the training/validation/test assignment.

Keep original animal identifiers, raw videos, and precise facility details in the authorized research environment. This repository intentionally contains **no real data manifest**. A public example, if added later, should use synthetic paths and observation identifiers.

## Relationship to the GUI

The Windows GUI reads an already prepared image or video and predicts leftover weight; it is **not** responsible for creating the controlled-weight reference dataset or performing study-level splitting. The GUI's default **1.00-s** sampling and **no-rotation** processing are deployment settings for speed, not confirmed retrospective statements of the original experiment.

## Verification required before publication

- Confirm the final controlled-weight image counts and whether multiple images correspond to a shared observation.
- Document the two independent-test exclusions (92 initially, 90 analyzed) and recalculate the final reference-weight distribution.
- Reconcile internal records of clip windows and actual saved clip lengths.
- Identify the precise timestamp-alignment rule used for each observation.
- Document camera positioning and acquisition conditions from original laboratory records.
- Keep test observations excluded from parameter tuning, early stopping, and checkpoint selection.
