# Lessons Learned: v1 to v2

## v1 (previous attempt)
- 3,000 samples (about 2% of one file), channels 0-3, target `m`.
- CNN: MAE 103.89, RMSE 119.82, R² -0.0007. FC: MAE 99.02, R² 0.057.
- Baseline (always predict the mean), validation: MAE 103.71, RMSE 119.74, R² -0.0001.
- The v1 CNN matched the baseline, so it had learned nothing.

## Data lessons
- The dataset is large. v2 uses 20,000 samples taken from 20 positions in the file.
- Don't load everything into memory. Read in batches (`pyarrow` `iter_batches`), convert a subset once, and save it as a compact float16 `.npz` on Google Drive.
- Colab's local disk is temporary. Keep data and checkpoints on Drive.
- The target `m` is uniform from 85 to 500, so scaling by 500 is enough.
- Two pixels in channel 0 overflowed float16 (`inf`). Clip before casting. CMSSW preprocessing will need the same clipping.

## Training lessons (from the reference ResNet work)
1. Normalize each input channel to 0-1 and clip outliers.
2. Check the target distribution, and scale or reshape it if it is skewed.
3. Use MAE as the loss. Report MAE, RMSE, R² and MRE.
4. Use a reduce-on-plateau learning rate scheduler.
5. Use all four allowed channels.
6. Write a small ResNet myself from the structure, not copy.
7. Use a validation set for all decisions. Touch the test split once, at the end.
8. Fix seeds and save the split.
9. Decide early whether to use `ieta`/`iphi`, because it affects the ONNX export.

Reference result: ResNet-15 on 2 million samples, MAE 56.43 GeV, MRE 24.04%. Not directly comparable to a 20k subset.

## Goals before emailing
1. A ResNet that clearly beats the baseline (MAE, MRE, R² on validation and test).
2. An ONNX export checked against PyTorch outputs.
3. A real CMSSW inference attempt. If it fails, a log of the exact command and error.

Two numbers matter most: the improvement over the baseline, and the train/validation gap.

## v2 results (to fill in)
| Model | MAE | RMSE | R² (val) | R² (test) |
|---|---|---|---|---|
| Baseline | 103.71 | 119.74 | -0.0001 | |
| v2 ResNet | | | | |