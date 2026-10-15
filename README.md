# Polyp Segmentation in Colonoscopy Frames with U-Net

A U-Net (TensorFlow/Keras) that segments polyps pixel by pixel in colonoscopy frames from the CVC-ClinicDB dataset. This is a learning project: the model is undertrained and its results are weak (see Results and Limitations). It is not a medical device.

## Data

[CVC-ClinicDB](https://polyp.grand-challenge.org/CVCClinicDB/): colonoscopy frames (`Original/*.tif`) with polyp masks (`GroundTruth/*.tif`). The dataset is not included here; download it from the link above. Its terms restrict use to research and education, and commercial use is forbidden. The weights in `files/model.h5` were trained on it, so the same restriction applies to them.

If you use the data, cite: Bernal J., Sánchez F. J., Fernández-Esparrach G., Gil D., Rodríguez C., Vilariño F. (2015). WM-DOVA maps for accurate polyp highlighting in colonoscopy: Validation vs. saliency maps from physicians. *Computerized Medical Imaging and Graphics*, 43, 99–111.

## Method

- Images and masks resized to 256 × 256, scaled to [0, 1].
- Split 368 / 122 / 122 (train / validation / test) with `train_test_split(random_state=42)`. Frames were split at random, not by video sequence.
- U-Net, binary cross-entropy, Adam (lr 1e-4), batch 8, 20 epochs, `ReduceLROnPlateau`. Training logs: `files/data.csv`, TensorBoard logs in `logs/`.

## Results

Test set (122 frames), from `Notebook.ipynb` cell 17:

| Metric | Value |
| --- | --- |
| Pixel accuracy | 0.932 |
| Recall | 0.495 |
| Precision | 0.712 |
| Soft IoU (see note) | 0.182 |

Final-epoch training vs validation (from `files/data.csv`): recall 0.836 vs 0.461 and soft IoU 0.297 vs 0.164, so the model overfits.

**Note on IoU.** The `iou` metric in the notebook multiplies the mask by the predicted probabilities without thresholding them, so it is a "soft" IoU and is lower than the standard thresholded IoU (prediction > 0.5) used in papers. The standard IoU and Dice have not been computed yet. Pixel accuracy is high mainly because most pixels are background.

## Limitations and next steps

- Compute thresholded IoU and Dice on the test set; report per-image distributions, not only the mean.
- Split by video sequence so frames from one polyp are not in both train and test.
- Add augmentation and a Dice or Dice + BCE loss; train longer with early stopping on validation Dice.
- Compare with a published baseline on the same split before claiming any improvement.

## How to run

1. Download CVC-ClinicDB and place `Original/` and `GroundTruth/` under `CVC-ClinicDB/`.
2. `pip install tensorflow opencv-python scikit-learn numpy pandas tqdm`
3. Run `Notebook.ipynb` top to bottom.
