# Multi-Object Tracking with YOLO26x + BoT-SORT

**Course:** Introduction to Computer Vision (503080)  
**Dataset:** MOT17-02-FRCNN  
**University:** Ton Duc Thang University

---

## Demo Output

<img width="800" height="450" alt="Image" src="https://github.com/user-attachments/assets/576e4683-c9ea-4af3-ace7-31268d24ef54" />

---

## Overview

This project implements **Multi-Object Tracking (MOT)** using:
- **Detector:** YOLO26x (pretrained on COCO)
- **Tracker:** BoT-SORT with custom configuration
- **Dataset:** MOT17-02-FRCNN (MOT Challenge)
- **Evaluation:** TrackEval (HOTA, MOTA, IDF1)

---
## Quick Start

**1. Open the notebook in Google Colab**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Vhjny5sPQHMNM1lo519nVJVm18CA9T8U?usp=sharing)

**2. Install dependencies**

```bash
pip install ultralytics -q
pip install lap -q
pip install trackeval -q
```

**3. Mount Google Drive and set paths**

```python
from google.colab import drive
drive.mount('/content/drive')

BASE    = '/content/drive/MyDrive/CV_Final'
IMG_DIR = f'{BASE}/data/MOT17-02-FRCNN/MOT17-02-FRCNN/img1'
GT_DIR  = f'{BASE}/data/MOT17-02-FRCNN/MOT17-02-FRCNN/gt/gt.txt'
```

**4. Prepare the dataset**

Upload the `MOT17-02-FRCNN` folder to Google Drive at:
```python
MyDrive/CV_Final/data/MOT17-02-FRCNN/
```
**5. Run all cells sequentially**

| Cell | Description |
|------|-------------|
| Cell 1 | Install dependencies |
| Cell 2 | Mount Drive & set paths |
| Cell 3 | Verify dataset structure |
| Cell 4 | Run** YOLO26x** + **BoT-SORT** tracking, export video & .txt |
| Cell 5 | Patch **NumPy **compatibility for **TrackEval** |
| Cell 6 | Prepare **TrackEval** folder structure |
| Cell 7 | Evaluate ** HOTA / MOTA / IDF1** |

---

## BoT-SORT Configuration

Default `botsort.yaml` was replaced with a custom config tuned for MOT17-02-FRCNN:

```yaml
tracker_type: botsort
track_high_thresh: 0.2      # lowered to catch more detections in crowded scenes
track_low_thresh: 0.05      # recovery threshold for occluded objects
new_track_thresh: 0.4       # raised to reduce false/ghost tracks
track_buffer: 90            # 3 seconds at 30fps — handles long occlusions
match_thresh: 0.7           # relaxed for better re-ID after occlusion
fuse_score: true
gmc_method: sparseOptFlow   # global motion compensation
proximity_thresh: 0.5
appearance_thresh: 0.25
with_reid: false
```

---

## Experiments & Insights

Several experiments were conducted before arriving at the final configuration.
Key findings are documented below.

**Effect of image size (imgsz)**

MOT17-02-FRCNN has native resolution of 1920×1080. Running inference at the
default `imgsz=640` caused significant detail loss, especially for distant
pedestrians. Increasing `imgsz` improved DetA and MOTA substantially at the
cost of longer runtime.

| imgsz | HOTA | MOTA | IDF1 | Runtime |
|-------|------|------|------|---------|
| 640 (default) | ~30% | ~11% | ~31% | ~25 min |
| 1280 | ~34% | ~35% | ~43% | ~90 min |
| 1920 | 34.27% | 34.77% | 43.85% | ~3 hrs |

Final choice: `imgsz=1280` — best trade-off between quality and runtime.

**Effect of conf threshold**

The assignment requires `conf=0.3` as fixed. While lowering conf improves
Recall (more detections), it also increases false positives (CLR_FP). The
enforced `conf=0.3` is a deliberate constraint that limits Recall and
contributes to the lower MOTA compared to the original paper.

**Effect of track_buffer**

Increasing `track_buffer` from the default 30 to 90 reduced ID switches (IDSW)
from 153 to 143 by keeping lost tracks alive longer during occlusion. However,
setting it too high risks re-assigning the same ID to a different person after
a long absence.

**Effect of new_track_thresh**

Raising `new_track_thresh` from 0.25 to 0.4 reduced ghost tracks significantly,
bringing IDs (predicted) from 244 down to 232 while GT_IDs is only 62.

**Why results are lower than the original paper**

The original BoT-SORT paper reports HOTA 64.6% / MOTA 80.6% using a detector
fine-tuned on the full MOT17 training set with a dedicated FastReID model.
This project uses a COCO-pretrained YOLO26x without fine-tuning, as required
by the assignment. Fine-tuning on MOT17 would be expected to close this gap
significantly.

---

## Results

### Final Metrics (YOLO26x + Custom BoT-SORT)

> **Note:**The final metrics below represent the cumulative result of all
> optimizations combined (using imgsz=1280 along with the tuned track_buffer, new_track_thresh, and custom botsort.yaml).

| Metric | Value | Description |
|--------|-------|-------------|
| **HOTA** | **36.25%** | Overall performance |
| **MOTA** | **36.94%** | Detection accuracy |
| **IDF1** | **43.31%** | Association accuracy |
| IDSW | 143 | ID switches |
| CLR_FP | 1,090 | False positives |

### Comparison with Original BoT-SORT Paper

| Tracker | HOTA | MOTA | IDF1 |
|---------|------|------|------|
| BoT-SORT (paper) | 64.6% | 80.6% | 79.5% |
| **Ours (YOLO26x)** | **36.25%** | **36.94%** | **43.31%** |

> **Note:** The gap is expected because the original paper uses a detector
> fine-tuned specifically on MOT17, while this project uses a COCO-pretrained
> model as required by the assignment.

---

## Output Video & Source Code

| Resource | Link |
|----------|------|
| Output Video | [Google Drive](https://drive.google.com/drive/folders/1YCrAn-Zkrl9ttViKloY5BfcWx9RYos_z?usp=sharing) |
| Notebook | [Google Colab](https://colab.research.google.com/drive/1Vhjny5sPQHMNM1lo519nVJVm18CA9T8U?usp=sharing) |

---

## References

- [Ultralytics YOLO Tracking](https://docs.ultralytics.com/modes/track/)
- [BoT-SORT (NirAharon)](https://github.com/NirAharon/BoT-SORT)
- [TrackEval (JonathonLuiten)](https://github.com/JonathonLuiten/TrackEval)
- [MOT Challenge Dataset](https://docs.ultralytics.com/datasets/track/)
- [Introduction to MOT (Datature)](https://datature.io/blog/introduction-to-multiple-object-tracking-and-recent-developments)
- [LearnOpenCV - YOLO Tracking](https://learnopencv.com/yolov8-object-tracking-and-counting-with-opencv/)

---

## Authors

| Student ID | Name |
|------------|------|
| 52300088 | Lê Hoài An |
| 52300110 | Ngô Trọng Hữu |

---

## License

This project is for educational purposes only.
