# Multi-Object Tracking (MOT) Pipeline: YOLO11 & ByteTrack Analysis

## Project Overview
[cite_start]This repository contains a Multi-Object Tracking (MOT) pipeline developed for the **MOT17** benchmark[cite: 1, 2]. [cite_start]The project implements a "Tracking-by-Detection" architecture using **YOLO11n** as the feature extractor and **ByteTrack** for temporal association[cite: 3, 40].

[cite_start]As part of a research-oriented challenge, this implementation focuses on the trade-off between inference throughput (FPS) and identity persistence (IDF1), specifically addressing **Occlusion-Induced "Blinking"** and **Camera Dynamics**[cite: 16, 64, 75, 79].

## System Requirements & Setup
[cite_start]This pipeline was developed and benchmarked on **Apple M4 hardware** using Metal Performance Shaders (MPS) for hardware acceleration[cite: 22].

### 1. Environment
* [cite_start]**Hardware:** Apple Silicon (M4 tested)[cite: 22].
* [cite_start]**Performance:** Achieved **~85 FPS**, ensuring high **Temporal Continuity** for tracking[cite: 22, 18].
* [cite_start]**Key Dependencies:** `ultralytics`, `py-motmetrics`, `scipy`[cite: 17, 56].

### 2. Dataset Structure
[cite_start]Benchmarks were conducted across three key sequences: **MOT17-02, 05, and 09**[cite: 17]. 
* **Note:** Large datasets and video outputs are excluded from this repository via `.gitignore` to maintain repository hygiene.

```text
project-root/
├── Research and Implementation Report .pdf  # Final Technical Report
├── notebooks/
│   └── tracking_pipeline.ipynb              # Main Experimentation Logic
├── src/                                     # YOLO11 + ByteTrack Source Scripts
└── README.md                                # Project Documentation
```

## How to Reproduce Results
The entire experimental pipeline is self-contained within `notebooks/tracking_pipeline.ipynb`.

1. **Inference:** Run the initial cells to execute YOLO11n inference. The script will generate `.pkl` cache files locally to ensure that subsequent metric evaluations are fast and deterministic.
2. **Evaluation:** Once inference is complete, the evaluation block utilizes `py-motmetrics` to compute MOTA and IDF1 against the MOT17 Ground Truth.

## Quantitative Results
| Metric | Result | Analysis |
| :--- | :--- | :--- |
| **MOTA** | **~71.2%** | High localization accuracy using YOLO11n. |
| **IDF1** | **~78.4%** | Strong identity consistency across sequences. |
| **FPS** | **~85** | High throughput ensures minimal inter-frame displacement. |

## Qualitative Results: The Confidence Gap
To validate the **Adaptive Thresholding** logic, I compared the baseline against optimized settings in a high-density frame (**MOT17-02**).

* **Baseline (0.5 Threshold):** Captured only **5 pedestrians**. Background targets were suppressed, leading to track fragmentation.
* **Adaptive (0.2 Threshold):** Localized **14 pedestrians**. Recovering **9 additional targets** provided the "temporal hooks" necessary for ByteTrack to maintain identity through occlusions.

**Key Finding:** Lowering the threshold in crowded frames increased detection recovery by **180%**, directly reducing "blinking" and ID switches.

## Implementation & Reproducibility
1. **Architecture:** YOLO11n detector + ByteTrack association.
2. **Hardware:** Benchmarked on **macOS (M4)** using `device="mps"`.
3. **Execution:** Open `notebooks/tracking_pipeline.ipynb`. Inference results are cached locally as `.pkl` files for deterministic evaluation.
4. **Data Hygiene:** The MOT17 dataset and local caches are excluded via `.gitignore`.

## Research Challenges & Insights
Non-Deterministic Caching: During iterative testing, I observed that inference results could fluctuate due to model caching. I implemented a "fresh-state" predictor logic to ensure all comparative experiments were scientifically deterministic.

Precision-Recall Trade-off: While lowering the threshold to 0.15 recovered significant occlusions, it introduced a 2% increase in false positives (e.g., background noise identified as pedestrians). Future work would involve integrating a spatial mask to apply low thresholds only in known "crowd clusters."

## Deliverables
* **[Research and Implementation Report (PDF)](./MOT17_Research_Report.pdf)**: Detailed 5-step analysis covering dataset exploration, trade-offs, and failure mode diagnostics.
* **Source Code:** Documented pipeline logic within the `notebooks/` directory.

## Future Work
* **Global Motion Compensation:** To stabilize tracking during camera jitter.
* **Re-ID Integration:** Utilizing appearance features for long-term re-association.







