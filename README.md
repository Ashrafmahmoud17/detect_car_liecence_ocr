# 🚘 License Plate Recognition System
### YOLOv8 + Tesseract OCR | Real-Time Plate Detection & Text Extraction

---

## 📌 Overview

A real-time **Automatic License Plate Recognition (ALPR)** system that combines **YOLOv8** for plate detection and **Tesseract OCR** for text extraction. The system processes video input, reads plate numbers with a stabilization buffer, saves the annotated output video, and logs the most frequently detected plate to a text file.

The project has **2 versions** — a real-time stable display version and a final best-plate extraction version.

---

## 🧠 How It Works

```
Video Input
     │
     ▼
YOLOv8 (license_plate_detector.pt)
     │
     ▼
Crop Plate Region
     │
     ▼
Preprocessing:
  ├── Grayscale
  ├── Resize ×2
  ├── Gaussian Blur (v1) / Threshold (v2)
  └── Binary Threshold
     │
     ▼
Tesseract OCR (--oem 3 --psm 7)
     │
     ▼
Clean Text → Filter (len >= 5)
     │
     ├── V1: Buffer (deque 10) → Most Common → Display on Frame → Save Video
     └── V2: Collect All → Counter → Most Frequent → Save to final_plate.txt
```

---

## 📁 Project Structure

```
license-plate-recognition/
│
├── v1_realtime_stable.py        # Real-time recognition with stability buffer
├── v2_best_plate.py             # Full video scan → extract most frequent plate
│
├── license_plate_detector.pt    # Custom YOLOv8 plate detection model
├── videoplayback.mp4            # Input video
│
├── output_result.mp4            # Annotated output video (v1)
└── final_plate.txt              # Most frequently detected plate number (v2)
```

---

## ⚙️ Requirements

### Python Version
```
Python 3.8+
```

### Install Libraries
```bash
pip install ultralytics
pip install pytesseract
pip install opencv-python
```

### Install Tesseract OCR
Download and install from:
👉 https://github.com/UB-Mannheim/tesseract/wiki

Default install path:
```
C:\Program Files\Tesseract-OCR\tesseract.exe
```

> Make sure the path in the code matches your installation.

---

## 🚀 Version Guide

### Version 1 — Real-Time Stable Display (`v1_realtime_stable.py`)

Uses a **rolling buffer** of the last 10 OCR readings to display the most consistent plate text — reduces flickering from noisy OCR frames.

```python
buffer = deque(maxlen=10)

# Add new reading
buffer.append(text)

# Get most common reading
stable_text = Counter(buffer).most_common(1)[0][0]
```

**Output:**
- Live window with green bounding box and stable plate text
- Saved annotated video: `output_result.mp4`

---

### Version 2 — Best Plate Extraction (`v2_best_plate.py`)

Scans the **entire video**, collects all OCR readings, then selects the most frequently occurring plate number as the final result.

```python
all_plates = []
# ... process all frames ...
final_plate = Counter(all_plates).most_common(1)[0][0]
```

**Output:**
- Printed to console: `Most Frequent Plate: ABC1234`
- Saved to file: `final_plate.txt`

---

## 🔧 Preprocessing Pipeline

```python
gray  = cv2.cvtColor(plate, cv2.COLOR_BGR2GRAY)   # Grayscale
gray  = cv2.resize(gray, None, fx=2, fy=2)         # Upscale ×2
gray  = cv2.GaussianBlur(gray, (5,5), 0)           # Reduce noise (v1 only)
_, thresh = cv2.threshold(gray, 140, 255,
            cv2.THRESH_BINARY)                      # Binarize
```

---

## 🎛️ Configuration

```python
# OCR Engine
config = '--oem 3 --psm 7'
# oem 3 = Default LSTM engine
# psm 7 = Single line of text (ideal for plates)

# Minimum valid plate length
if len(text) >= 5:      # v1
if len(text) >= 4:      # v2

# Stability buffer size (v1)
buffer = deque(maxlen=10)

# Input / Output
cap = cv2.VideoCapture("videoplayback.mp4")
out = cv2.VideoWriter("output_result.mp4", ...)
```

---

## 📊 Comparison: V1 vs V2

| Feature | V1 — Real-Time | V2 — Best Plate |
|---|---|---|
| Display | Live window | No display |
| Stability | Rolling buffer (10 frames) | Full video voting |
| Output | Annotated video | `final_plate.txt` |
| Use case | Monitoring / surveillance | Single vehicle identification |
| Speed | Real-time | Slower (full scan) |
| Accuracy | Good | Higher |

---

## 💡 Tips for Better OCR Accuracy

- Use a **high-resolution** input video (720p+)
- Ensure **good lighting** — avoid shadows on the plate
- Adjust the **threshold value** (`140`) based on plate brightness:
  - Dark plates → lower value (e.g. `120`)
  - Bright plates → higher value (e.g. `160`)
- For Arabic/non-Latin plates, configure Tesseract language:
  ```python
  config = '--oem 3 --psm 7 -l ara'
  ```

---

## ⚠️ Known Limitations

- **No vehicle tracking** — each frame is processed independently
- **OCR accuracy** depends heavily on plate image quality and resolution
- **Tesseract** struggles with stylized fonts or dirty plates
- Works best with **standard rectangular license plates**

---

## 👤 Author

**Ashraf Mahmoud**
Computer Sciences — New Mansoura University

---

## 📄 License

This project is for educational purposes.
