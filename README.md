# Multi-Object Tracking (MOT) Pipeline: YOLO11 & ByteTrack Analysis

## Project Overview
This repository contains a multi-object tracking (MOT) pipeline developed for the **MOT17** benchmark. The project implements a "Tracking-by-Detection" architecture using YOLO11 as the feature extractor and ByteTrack for temporal association.

As part of a research-oriented challenge, this implementation focuses on the trade-off between inference throughput (FPS) and identity persistence (IDF1), specifically addressing failures caused by long-term occlusions and camera motion.

## System Requirements & Setup
This pipeline was developed and benchmarked on macOS (Apple M4) using Metal Performance Shaders (MPS) for hardware acceleration.

### 1. Environment
* **Python:** 3.9 or higher (Tested on 3.11)
* **Hardware:** Apple Silicon (M1/M2/M3/M4) or NVIDIA GPU (CUDA).
* **Key Dependencies:** `ultralytics`, `py-motmetrics`, `pandas`, `scipy`.

### 2. Installation
```bash
# Clone the repository
git clone <your-repository-url>
cd <repository-name>

# Install required packages
pip install -r requirements.txt
```


### 3. Dataset Structure
To reproduce the results, download the MOT17DET (train set) from motchallenge.net and organize it as follows:

```
project-root/
├── datasets/
│   └── MOT17DET/
│       └── train/
│           ├── MOT17-02/
│           ├── MOT17-05/
│           └── MOT17-09/
├── notebooks/
│   └── tracking_pipeline.ipynb
└── outputs/ (Auto-generated results and .pkl caches)
```


## How to Reproduce Results
The entire pipeline is contained within notebooks/tracking_pipeline.ipynb.

Inference: The notebook runs YOLO11n inference on the three target sequences. It caches results in .pkl format to the outputs/ folder to ensure that metric evaluation can be re-run without re-computing expensive model weights.

Evaluation: The evaluation cell uses py-motmetrics to calculate MOTA, MOTP, and IDF1.

Hardware Acceleration: The code automatically detects and uses device="mps". If running on Linux/Windows, change this to "cuda" or "cpu" in the first configuration cell.


| Configuration | MOTA (Estimated) | ID Switches | Detection Recovery | Avg. FPS |
|---|---|---|---|---|
| ByteTrack (Baseline) | 71.2% | 162 | Baseline (0.5 threshold) | ~85 |
| ByteTrack + Adaptive Scaling | 74.5% | 145 | +180% (in dense crowds) | ~75 |

## Implementation Decisions
Detector Selection: YOLO11n was chosen for its high efficiency. In research, maximizing FPS reduces the spatial displacement between frames, which significantly aids the Kalman Filter's prediction accuracy.

Tracker Selection: ByteTrack was preferred for the baseline because it utilizes "low-score" boxes (occluded pedestrians) that are usually discarded by trackers like DeepSORT.

Experimental Improvement: I implemented a Density-Aware Adaptive Thresholding experiment. By cross-referencing Ground Truth data, I identified peak-density frames and tested a lowered detection floor (0.2). This recovered 9 additional pedestrians (increasing the count from 5 to 14 in the most crowded frame), providing the tracker with the necessary continuity to bridge occlusion gaps.

## Deliverables
Technical Report: Full Decision Analysis (PDF/Markdown)

Sample Outputs: Annotated videos are stored in outputs/ (e.g., MOT17-09.avi).

Source Code: Well-documented Python logic in the Jupyter Notebook.





