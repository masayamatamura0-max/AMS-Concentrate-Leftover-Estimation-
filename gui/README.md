# AMS Leftover Estimator — Windows GUI user guide

[← Repository homepage](../README.md) · [Research methods](../README.md#materials-and-methods)

This page explains how to install and use the **precompiled Windows GUI** to estimate concentrate leftovers from AMS feed-bowl images or short video clips. The application uses trained Mask R-CNN and EfficientNet V2-S checkpoints. It **does not train the models**.

## 1. Download the application

1. Open **[the latest GitHub Release](../../../releases/latest)**.
2. Under **Assets**, download `AMS-Leftover-Estimator-app.zip`. **Do not** use GitHub's auto-generated *Source code (zip)* file.
3. Extract the entire archive to a folder on your Windows computer. Do not run the application directly from inside the compressed ZIP.

The extracted folder should contain files resembling the following:

```text
AMS-Leftover-Estimator-app/
├── AMS-Leftover-Estimator.exe
├── _internal/
├── Model/
│   ├── Detectron2.pth
│   └── EfficientNet.pth
└── data/
    ├── Clips/
    └── Cropped images/
```

- Keep **`_internal/`** alongside the EXE. It holds libraries and other files needed to run the packaged program.
- Keep the **`Model/`** folder available to the application. The release includes the two trained `.pth` checkpoints, so a separate download is not normally necessary.
- The **`data/`** folder contains **selected example files only**, not all images or observations analyzed in the study.
- A separate Python installation is **not** required to run the packaged EXE.

## 2. Start the GUI and check the models

Double-click `AMS-Leftover-Estimator.exe`.

The **Model input** indicator summarizes whether model checkpoint files were found automatically. Click the indicator to view the detected paths or access advanced model settings when needed. For normal use, you do not have to select the checkpoints manually if they are found in the packaged `Model/` folder.

Model discovery and actual model initialization are separate: the application may report a model as found before its weights have been loaded into memory. The **first prediction may take longer** while deep-learning libraries and model weights are initialized.

## 3. Select an input mode

Choose one of the two available modes:

| Input mode | Use it for | What happens |
| --- | --- | --- |
| **Crop or Clip** (default) | An already-cropped feed-bowl image **or** a short video clip | Cropped image → EfficientNet directly; clip → Detectron2 selects the best accepted bowl crop → EfficientNet |
| **Full image** | A still image of the broader AMS feeding area | Detectron2 locates the bowl and provides a crop → EfficientNet |

For a video clip, candidate frames are sampled at the interval set in the GUI (**1.00 s by default**). The accepted detection with the highest detector score is used to provide **one crop** for **one** EfficientNet prediction; the app does not average predictions across frames.

## 4. Open an image or clip and make a prediction

1. Click the input-file selection control. The dialog initially opens in the folder containing the EXE.
2. Choose a supported image or video file consistent with the selected input mode.
3. **Detection and/or prediction begins automatically** after a file is selected. You do not need to press a separate Start button for the initial analysis.
4. Read the **estimated concentrate leftover weight (g)** and inspect the preview images.
5. If you change the relevant settings, click **Detect / Predict** to run the analysis again.

### Preview behavior

- **Cropped still image:** the feed-bowl crop is presented as one large image.
- **Full image or video clip:** the GUI shows the original image/best frame, a detection overlay, and the selected feed-bowl crop.

The model output is a predicted leftover **mass in grams**. A detector confidence score indicates confidence in locating a bowl; it is not a confidence interval for the predicted mass.

## 5. Troubleshooting

| Problem | What to check |
| --- | --- |
| Model not found | Confirm that `Model/Detectron2.pth` and `Model/EfficientNet.pth` are in the extracted application folder; inspect the Model input details. |
| EXE does not launch | Make sure the full ZIP was extracted and `_internal/` was not moved or removed. |
| No accepted bowl crop | Check image visibility, the selected input mode, and the detector settings. |
| Video does not open | Try a compatible MP4 file; decoding depends on the formats supported by the bundled OpenCV build. |
| First prediction is slow | This can occur when loading the trained checkpoints and runtime libraries for the first time. |
| Unexpected weight estimate | Check that the visible bowl and concentrate resemble the conditions used for evaluation. The controlled-weight development range was 0–1,000 g. |

## Scope and data availability

The GUI is distributed for **inference using trained checkpoints**. Its release ZIP includes executable files and model weights, not the Python training code or the complete experimental dataset. Scientific details are available from the homepage's [Materials and Methods](../README.md#materials-and-methods) links.
