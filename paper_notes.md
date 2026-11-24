# Paper Notes: End-to-end deep learning inference with CMSSW via ONNX using Docker
arXiv:2309.14254

## Summary
- Integrates a deep learning pipeline for particle identification, using low-level detector data from high-energy collisions, into the CMS software framework (CMSSW).
- Covers the model, the pipeline and its performance (latency and events per second) on CPUs and NVIDIA GPUs.
- Integration at NERSC was done by preparing a CMSSW Docker image.

## Introduction
- Earlier ML helped physics but failed at some identification tasks, so more advanced deep learning (CNNs, graph neural networks) is needed.

## Dataset
- The data is simulated, not real collisions. Software first simulates what happens when particles collide, then how the CMS detector responds.
- In simulation the true answer for every particle and event is known, so deep learning can be trained on it.
- Taggers are classifiers that identify which particle or jet it is.
- The input has multiple channels: each sub-detector measures a different aspect.

## Pipeline (in my words)
Event data lives in an EDM ROOT file. Every stage reads from it and writes its output back for the next stage.

1. **FrameProducer, part 1 (detector data producers):** turns 1D detector hits into whole-event images, one per sub-detector (channel), and stores them back.
2. **FrameProducer, part 2 (object-level producers):** finds objects using seeds and crops frames around them. `EGFrameProducer` handles electrons and photons; `JetFrameProducer` handles quark/gluon, top and tau jets.
3. **Taggers (inference):** take the cropped frames with all channels and feed them to the model through ONNX Runtime (supported by CMSSW, including on GPU). Predictions are written back to the EDM file.

Code packages: `DataFormats`, `FrameProducers`, `Taggers`.

## Docker and containerization
- Installing CMSSW on a machine without a CMS setup is difficult, so CMSSW and its dependencies were packaged in a Docker image. It runs the same everywhere, including NERSC's Perlmutter.
- **CVMFS:** the file system that distributes CMS software. The container needs it.
- **Shifter:** NERSC's tool for running Docker images.
- The paper used CMSSW 12_0_6. For my task I follow the evaluation test: `clelange/cmssw`, tag 10_6_8_patch1.

## Performance
- Measured with latency and throughput.
- The model is a small part of the runtime. Most of the cost is I/O, because the output file stores much more than the prediction. Writing only the required information would help.
- For me: a regression model adds almost no time. What matters is that it runs correctly, with the right input shape and preprocessing, and that the output is saved properly.

## Conclusion and where my project fits
- The framework integrates with CMSSW, supports four taggers, runs on CPU and GPU, and works on NERSC through the Docker image.
- There is no separate future-work section, and the paper does not mention regression.
- All four existing taggers are classifiers. The E2E repo README says regression is "to come soon". My task is to add regression model inference.

## Questions for the mentors
- Is there a recommended CMSSW Docker image or setup for a student to reproduce the inference pipeline locally?
- For regression, should the output be saved as a new data format in `DataFormats`, or reuse the existing tagger output structure?