# 🎯 Multi-Object Tracking with YOLO26x + BoT-SORT

**Course:** Introduction to Computer Vision (503080)
**Dataset:** MOT17-02-FRCNN
**University:** Ton Duc Thang University

---

## 🎬 Demo Output
https://github.com/HoaiAn001/CV_TDTU_Final/assets/output_web.mp4

---

## 📋 Overview

This project implements **Multi-Object Tracking (MOT)** using:
- **Detector:** YOLO26x (pretrained on COCO)
- **Tracker:** BoT-SORT with custom configuration
- **Dataset:** MOT17-02-FRCNN (MOT Challenge)
- **Evaluation:** TrackEval (HOTA, MOTA, IDF1)

---

## 🚀 Quick Start (Google Colab)

### 1. Open Notebook
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/Y)
---

## 📊 Results

### Metrics Comparison

| Model | HOTA | MOTA | IDF1 | IDSW |
|-------|------|------|------|------|
| YOLOv8s | 30.47% | 11.18% | 31.37% | 49 |
| YOLO26n | 33.68% | 26.87% | 38.69% | 40 |
| YOLO26x (imgsz=1920) | 34.27% | 34.77% | 43.85% | 153 |
| **YOLO26x + custom yaml** | **36.25%** | **36.94%** | **43.31%** | 143 |

### Comparison with Original BoT-SORT Paper

| Tracker | HOTA | MOTA | IDF1 |
|---------|------|------|------|
| BoT-SORT (paper) | 64.6% | 80.6% | 79.5% |
| **Ours (YOLO26x)** | **36.25%** | **36.94%** | **43.31%** |

> **Note:** The gap is expected because the original paper uses a detector
> fine-tuned specifically on MOT17, while this project uses a COCO-pretrained
> model as required by the assignment.

---

## 🎬 Output Video

> 📎 [https://drive.google.com/drive/folders/1YCrAn-Zkrl9ttViKloY5BfcWx9RYos_z?usp=sharing](#) ← thay link Drive của bạn vào đây

---

## 📖 References

- [Ultralytics YOLO Tracking](https://docs.ultralytics.com/modes/track/)
- [BoT-SORT (NirAharon)](https://github.com/NirAharon/BoT-SORT)
- [TrackEval (JonathonLuiten)](https://github.com/JonathonLuiten/TrackEval)
- [MOT Challenge Dataset](https://docs.ultralytics.com/datasets/track/)
- [Introduction to MOT (Datature)](https://datature.io/blog/introduction-to-multiple-object-tracking-and-recent-developments)
- [LearnOpenCV - YOLOv8 Tracking](https://learnopencv.com/yolov8-object-tracking-and-counting-with-opencv/)

---

## 👥 Authors

| Student ID | Name |
|------------|------|
| MSSV | Họ và tên |
| 52300088 | Lê Hoài An |
| 52300110 | Ngô Trọng Hữu |
---

## 📄 License

This project is for educational purposes only.
