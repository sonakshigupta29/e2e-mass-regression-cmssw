# e2e-mass-regression-cmssw

# Particle mass regression for CMS E2E (ML4SCI GSoC test, Tasks 2f and 2g)

**Status:** data pipeline and baseline are done. ResNet training and CMSSW inference are in progress.

## Goal
- **Task 2f:** train a model that regresses particle mass from 125x125 jet images, 80/20 train/test split.
- **Task 2g:** export the model to ONNX and run inference inside CMSSW (Docker).

## Data
- Source: `top_gun_opendata_0.parquet` from the test's CERNBox link (150,327 samples in this file).
- Used here: 20,000 samples taken from 20 evenly spaced positions in the file.
- Input: channels 0-3 of the 8-channel image (Track pT, two impact-parameter channels, ECAL). The channel identity is inferred from value ranges and plots, not from documentation.
- Target: mass `m`, uniform between 85 and 500, scaled by 500 for training.
- Split (seed 42): 14,000 train / 2,000 validation / 4,000 test. The test set is not used until the final evaluation.
- Data files are stored on Google Drive and are not in this repo.

## Baseline (validation)
Always predicting the mean mass gives MAE 103.71, RMSE 119.74, R2 about 0. A model has to beat this to count as having learned anything.

## Previous attempt (v1)
A CNN and a fully connected network trained on 3,000 samples reached MAE 103.89 (CNN) and 99.02 (FC), essentially the baseline. See `LESSONS.md`.

## Repo contents
- `notebooks/01_data_and_baseline.ipynb`: data inspection, subset build, split, baseline
- `LESSONS.md`: what went wrong in v1 and what changed
- `PAPER_NOTES.md`: notes on arXiv:2309.14254 (E2E framework with CMSSW and ONNX)
- `figures/`: target histogram and channel