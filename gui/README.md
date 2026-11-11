# Windows GUI — AMS Concentrate Leftover Estimator

[Back to repository homepage](../README.md) · [Scientific methods](../Materials_and_Methods/00_study_design_and_data.md)

This page is the **user manual** for the downloadable Windows application. It describes inference using **trained checkpoints**. **Python source code and Jupyter notebooks are not included in this repository.** The compiled application and trained checkpoints are available from GitHub Releases.

## 1. What the application does

The app estimates the mass of concentrate remaining in an AMS feed bowl from an image or short video. Two pretrained models are used as needed:

- **Detectron2 / Mask R-CNN:** locates the bowl in a full image or selected video frame.
- **EfficientNet V2-S:** predicts leftover mass (g) from the selected bowl crop.

An already cropped still image does not need object detection. The current GUI is an **inference** application; it cannot train or fine-tune either model.

## 2. Download and launch the Windows application

**Download:** Open the [latest Windows release](../../../releases/latest) and select `AMS-Leftover-Estimator-app.zip` under **Assets**. The distribution uses an `onedir` layout, avoiding unpacking a large one-file EXE at every launch. Building the executable from source is **not supported by the documentation in this repository**, because the source code is not published here.

1. Open the [latest release](../../../releases/latest) and download `AMS-Leftover-Estimator-app.zip` from **Assets**.
2. Extract the entire ZIP to a local folder.
3. Run `AMS-Leftover-Estimator.exe`. **Do not delete or move `_internal/` or `Model/` separately.**
4. Open one of the example files in `data/`, or select another compatible image or video. The included example files are **only a subset**, not the complete research dataset.

A large first-prediction delay can still occur because PyTorch/Detectron2 and the model weights are loaded lazily when inference begins. `onedir` primarily reduces repeated archive extraction at application startup.

### Folder layout in the Windows release ZIP

```text
AMS-Leftover-Estimator-app/
├── AMS-Leftover-Estimator.exe
├── _internal/                 # Required runtime dependencies
├── Model/
│   ├── Detectron2.pth        # Included model weight
│   └── EfficientNet.pth      # Included model weight
└── data/                     # Examples only; not all study data
    ├── Clips/
    └── Cropped images/
```

**Both trained checkpoints are included in this Windows release.** They do not need to be downloaded separately. Do not confuse the application examples under `data/` with the complete training or independent-evaluation dataset, which is not in the release. Users may load compatible image and video inputs from any accessible directory.

## 3. Automatic model selection

At startup and whenever an input file is chosen, the app searches for **exact filename matches**:

```text
Model/Detectron2.pth
Model/EfficientNet.pth
```

The search order is:

1. A `Model/` folder next to the **selected image/video**.
2. A `Model/` folder **one directory above** the input file's folder.
3. A `Model/` folder beside the **EXE**.

The two checkpoints are located independently, so they might come from different folders. Prefer keeping both matching checkpoints together and documenting the versions used.

The compact **Model input** indicator reports availability. Click it to expand checkpoint details. **Found** means the path exists; **Loaded** means inference has loaded the weights into memory. Advanced settings provide manual checkpoint selection, an optional matching **Detectron2 YAML**, and the detection confidence threshold. A checkpoint with an incompatible architecture or detector config will not become valid merely by renaming its file.

## 4. Choose the input mode

The GUI begins in **Crop or Clip** mode.

| Mode | Input | Required model(s) | Processing |
| --- | --- | --- | --- |
| **Crop or Clip** | Cropped bowl image | EfficientNet | Direct regression (no Detectron2) |
| **Crop or Clip** | Short video | Detectron2 + EfficientNet | Sample frames → best-confidence crop → one regression |
| **Full image** | Uncropped still image | Detectron2 + EfficientNet | Detect bowl → crop → regress |

**Supported input extensions** are `.jpg`, `.jpeg`, `.png`, `.bmp`, `.tif`, `.tiff`, `.webp` for images and `.mp4`, `.avi`, `.mov`, `.mkv`, `.m4v` for videos. Available video codecs depend on the OpenCV runtime. Videos are only valid in **Crop or Clip** mode.

### Open a file and predict

1. Select **Crop or Clip** or **Full image**.
2. Click **Open image / clip…** (or **Open full image…**).
3. Navigate to the file and click **Open**. The file picker initially displays the folder where the EXE is located.
4. **Prediction starts automatically** after a supported file is selected.
5. Read the predicted mass in **grams** and inspect the displayed image(s).

The first prediction can be slower than subsequent ones because the models load on demand. Model-loading progress is not the same as a completed prediction.

## 5. Video handling

For video clips, the current GUI:

- Samples at **1.00-s intervals by default**.
- Uses frames in their **original orientation**.
- Runs Detectron2 **once on each sampled frame**.
- Chooses the crop with the **highest accepted detector confidence** (default confidence threshold **0.90**).
- Applies EfficientNet **once** to that selected crop.

A 4-second, 20-fps video typically yields sampled frames at approximately **0, 1, 2, and 3 seconds**. Changing the interval changes the number of detector candidates. The GUI does **not** average EfficientNet predictions over the clip.

**Scientific caution:** The original study records include other frame-sampling intervals and potentially rotated detector candidates. Those research files are not included here. The GUI's faster default is not automatically equivalent to the experimental pipeline used to evaluate scientific performance.

## 6. View and save predictions

The lower part of the window contains the **Prediction** result area:

- **Cropped still image:** one large **Feed-bowl crop** preview.
- **Video clip:** three previews — best original frame, detection overlay, and selected feed-bowl crop.
- **Full image:** three previews — original image, detection overlay, and crop.

The **Detect / Predict** button has its standard appearance and can rerun the selected file after a setting changes. **Save crop & overlay…** exports `feed_bowl_crop.png`; when a detection was performed, it also exports `detection_overlay.png`.

The app displays a **point prediction**, not a quantified confidence interval. The detector confidence displayed in a status message is not an uncertainty interval for predicted grams.

## 7. Advanced settings

Expand **Model input** and then **Advanced settings** to:

- Select an alternative **EfficientNet `.pth`** or **Detectron2 `.pth`** file manually.
- Select or clear the detector's **YAML** configuration where the checkpoint requires one.
- Adjust **minimum detection confidence** (default `0.90`).
- Return to automatic checkpoint discovery if a manual selection is no longer appropriate.
- Adjust **Video sample every (s)** when a clip is selected (default `1.00`, current valid range `0.05–10.00`).

If a threshold or sampling interval is changed, press **Detect / Predict** to rerun the existing input; selecting a different supported input launches a new prediction automatically.

## 8. Troubleshooting

| Symptom | Likely cause | What to check |
| --- | --- | --- |
| Model not found | Missing or differently named checkpoint | Verify `Model/EfficientNet.pth` or `Model/Detectron2.pth` and the search order |
| Cropped image cannot be predicted | EfficientNet missing/incompatible | Use an authorized checkpoint matching EfficientNet V2-S |
| Full image/video cannot be detected | Detectron2 missing/incompatible | Verify detector checkpoint, YAML, threshold, and input content |
| Clip cannot be opened | Unsupported or unavailable video codec | Try a standard H.264 MP4 or validate decoding with the installed OpenCV build |
| GUI opens but first prediction is slow | Heavy ML libraries/weights are loading | Subsequent predictions may be faster; `onedir` mainly accelerates application startup |
| No bowl crop found | Low score, occlusion, wrong framing | Inspect source frame, model compatibility, and detection confidence |
| EXE fails after moving files | `_internal/` was not moved | Keep the entire extracted `onedir` distribution together |
| Crop estimate is implausible | Different bowl/domain, incompatible weight checkpoint, unusual lighting | Check image content, reference range, and correct training checkpoint |

A detection failure should be reported as a failure, **not** as 0 g of feed.

## 9. Research-method differences

A useful distinction for collaborators is:

| Scientific workflow | Deployed GUI |
| --- | --- |
| Internal research records contain `0.05 s` and `0.20 s` sampling options (final executed interval requires verification) | **1.00 s** default |
| Research records may include rotated detector candidates | **Original orientation only** |
| Controlled-weight training data, hyperparameter optimization | **Inference only** |
| Internal validation and separate natural-leftover evaluation | **Individual-file predictions** |

See [Object detection](../Materials_and_Methods/01_object_detection.md), [Leftover-weight estimation](../Materials_and_Methods/02_leftover_weight_estimation.md), and [Evaluation](../Materials_and_Methods/03_model_evaluation.md) for method details.
