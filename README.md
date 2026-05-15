# Multi-Object Tracking — YOLO26x + BoT-SORT

**Course:** Introduction to Computer Vision (503080)  
**Dataset:** MOT17-02-FRCNN  
**University:** Ton Duc Thang University

---

## Demo Output

<img width="800" height="450" alt="Image" src="https://github.com/user-attachments/assets/75f3b0fd-205d-41fb-949d-4c95d9bd0752" />

---

## Overview

Multi-Object Tracking pipeline built on top of YOLO26x and BoT-SORT, with post-processing from StrongSORT (TPAMI 2023).

| Component | Details |
|-----------|---------|
| Detector | YOLO26x pretrained on COCO |
| Tracker | BoT-SORT with custom configuration |
| Post-processing | AFLink + Trajectory Interpolation |
| Dataset | MOT17-02-FRCNN (600 frames, 62 pedestrians) |
| Evaluation | TrackEval — HOTA, MOTA, IDF1 |

---

## Quick Start

**1. Open in Google Colab**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1bnvqfdLam_4LKBMs0lmNrpd0JJJJ5Bhp?usp=sharing)

**2. Upload dataset to Google Drive**

```
MyDrive/CV_Final/
├── data/
│   ├── img1/          ← frames from MOT17-02-FRCNN
│   └── gt/gt.txt      ← ground truth
└── results/           ← auto-created
└── trackeval/         ← contain results of metrics
```

**3. Run all cells sequentially**

| Cell | Description |
|------|-------------|
| 1 | Install dependencies |
| 2 | Mount Google Drive |
| 3 | Define paths and verify dataset |
| 3.5 | Configure BoT-SORT tracker |
| 4 | Run YOLO26x + BoT-SORT, export video and `.txt` |
| 4.5 | Post-processing: AFLink + Trajectory Interpolation |
| 5 | Prepare TrackEval folder structure |
| 6 | Evaluate HOTA / MOTA / IDF1 |
| 7 | Display output video |

---

## BoT-SORT Configuration

Custom `botsort.yaml` tuned for MOT17-02-FRCNN (fixed camera, crowded pedestrian scene):

```yaml
tracker_type: botsort
track_high_thresh: 0.3      # confidence threshold for high-quality detections
track_low_thresh: 0.05      # recovery threshold for occluded objects
new_track_thresh: 0.6       # minimum confidence to initialize a new track
track_buffer: 90            # 3s at 30fps — handles long occlusions
match_thresh: 0.8           # IoU threshold for detection-to-track matching
fuse_score: true
gmc_method: sparseOptFlow   # camera motion compensation (better than ECC on fixed cam)
proximity_thresh: 0.4
appearance_thresh: 0.35
with_reid: false
```

> `gmc_method: sparseOptFlow` was selected over `ecc` after ablation — ECC degraded IDF1 by 3.83pp on this fixed-camera sequence.

---

## Post-processing

Two techniques from StrongSORT (TPAMI 2023) applied after tracking:

**AFLink (Appearance-Free Link)**  
Links fragmented tracklets using spatial-temporal cost and Hungarian assignment. Reduces spurious IDs and fragmentation without additional model inference.

**Trajectory Interpolation**  
Fills detection gaps up to 20 frames via linear interpolation on bounding box coordinates. Reduces false negatives and improves MOTA.

Both steps operate on the `.txt` result file in-place — no extra GPU inference needed.

---

## Ablation Study

| Configuration | HOTA | MOTA | IDF1 | IDs | IDSW |
|---------------|------|------|------|-----|------|
| Baseline — YOLO26n · 640px | 26.67% | 18.67% | 26.67% | — | — |
| Exp 1 — YOLO26m · 640px | 33.68% | 26.87% | 38.69% | — | — |
| Exp 2 — YOLO26x · 1280px | 34.61% | 34.79% | 43.56% | 255 | 163 |
| Exp 3 — + AFLink + Interp (sparseOptFlow) | **36.91%** | **40.87%** | **44.36%** | **35** | **79** |
| Exp 4 — Exp 3 with ECC instead | 35.72% | 40.07% | 39.73% | 33 | 93 |

Exp 3 is the final configuration. The drop in Exp 4 confirms that `sparseOptFlow` is better suited for fixed-camera sequences like MOT17-02.

---

## Results

### Final Metrics

| Metric | Value | Description |
|--------|-------|-------------|
| **HOTA** | **36.91%** | Overall performance (detection + association) |
| **MOTA** | **40.87%** | Detection accuracy |
| **IDF1** | **44.36%** | Identity consistency |
| IDSW | 79 | ID switches |
| IDs | 35 | Predicted track IDs (GT: 62) |
| Frag | 129 | Fragmented tracks |

### Comparison with Original Paper

| Method | HOTA | MOTA | IDF1 |
|--------|------|------|------|
| BoT-SORT (paper, fine-tuned) | 64.6% | 80.6% | 79.5% |
| **Ours (COCO pretrained, no fine-tune)** | **36.91%** | **40.87%** | **44.36%** |

The gap is expected — the original paper uses a detector fine-tuned on MOT17 with FastReID, while this project uses a COCO-pretrained model as required by the assignment.

---

## Limitations & Future Work

**Limitations**

This project uses a COCO-pretrained YOLO26x without fine-tuning, as required
by the assignment. The domain gap between COCO (general objects) and MOT17
(crowded pedestrian scenes) limits Recall — particularly for small or heavily
occluded pedestrians — which directly caps MOTA and HOTA below the levels
reported in the original BoT-SORT paper.

Hardware constraints (Google Colab T4, no persistent sessions) also prevented
fine-tuning on the full MOT17 training set (~5.5GB, 8–10 hours of training).

**Future Work**

Two directions are proposed to push performance toward 50–65% HOTA:

1. **Fine-tune on CrowdHuman** — adapting the detector to dense pedestrian
   scenes would significantly improve Recall and reduce false negatives (CLR_FN),
   which currently stands at 9,433 missed detections out of 18,581 ground truth.

2. **Stronger ReID** — integrating a dedicated person ReID model trained on
   MSMT17 (e.g. OSNet) with proper weight loading would improve association
   accuracy after long occlusions, expected to increase IDF1 beyond the current
   44.36%.

---

## Output

| Resource | Link |
|----------|------|
| Output Video + Source | [Google Drive](https://drive.google.com/drive/folders/1mycrU134KojBY-65_ATMaEBx7nCWmXve?usp=sharing) |
| Notebook | [Google Colab](https://colab.research.google.com/drive/1bnvqfdLam_4LKBMs0lmNrpd0JJJJ5Bhp?usp=sharing) |

---

## References

- [Ultralytics YOLO Tracking](https://docs.ultralytics.com/modes/track/)
- [BoT-SORT — NirAharon](https://github.com/NirAharon/BoT-SORT)
- [StrongSORT — TPAMI 2023](https://github.com/dyhBUPT/StrongSORT)
- [TrackEval — JonathonLuiten](https://github.com/JonathonLuiten/TrackEval)
- [MOT Challenge Dataset](https://docs.ultralytics.com/datasets/track/)
- [ByteTrack](https://github.com/ifzhang/ByteTrack)

---

## Authors

| Student ID | Name |
|------------|------|
| 52300088 | Lê Hoài An |
| 52300110 | Ngô Trọng Hữu |

---
