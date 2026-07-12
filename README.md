# AUV Simulation for Underwater Pipeline Inspection

A computer vision pipeline that detects corrosion, tracks it frame to frame, and steers a simulated AUV along a subsea pipeline, built entirely inside Minecraft as a substitute for real underwater footage.

---

# Overview

Subsea pipeline inspection is usually carried out using divers or Remotely Operated Vehicles (ROVs). Both approaches are expensive, time-consuming, and limited by operating depth and weather conditions. Autonomous Underwater Vehicles (AUVs) provide a more scalable alternative, provided they can reliably perceive and follow underwater infrastructure.

Testing such a system on a real AUV was not practical for this student project, so the underwater environment and pipeline were recreated inside Minecraft. A custom world containing a corroded pipeline served as a stand-in for real subsea footage. The recorded gameplay was converted into image sequences, which were then used to train and evaluate computer vision models in the same way as real underwater sensor data.

---

# How it Works

The system processes the same video stream through two parallel pipelines.

## Corrosion Detection and Tracking

YOLOv8s detects corrosion patches in each frame. DeepSORT then associates detections across consecutive frames so that each corrosion patch maintains a consistent identity instead of being detected as a new object in every frame.

## Pipeline Segmentation and Navigation

HSV thresholding is used to separate the pipeline from the background. A minimum-area rectangle is fitted to the segmented contour to estimate the pipeline centreline and orientation. These estimates are then used to generate keyboard and mouse inputs through `pyautogui`, allowing the simulated AUV to remain centred on and aligned with the pipeline. A Hough Line Transform runs alongside the segmentation process as an additional edge verification step.

---

# Dataset

- 1,775 frames extracted from screen recordings of the simulated pipeline environment
- Annotated in Roboflow using a single class: **corrosion**
- Augmented to approximately 3,500 images using ±15° rotations
- Split into 70% training, 20% validation, and 10% testing

---

# Training and Results

The corrosion detector was trained using YOLOv8s with an input resolution of 640 × 640 pixels, a batch size of 16, and a scheduled training duration of 120 epochs. Early stopping was triggered at epoch 78. At that point, recall had reached approximately 0.50, indicating that some corrosion patches were still being missed, while precision and mAP@0.5 remained stable throughout training. Since additional epochs produced little improvement, training was stopped before completing the full schedule.

HSV-based segmentation performed reliably under consistent lighting conditions, although occasional false positives occurred when rust regions closely matched the background colour. DeepSORT maintained stable identities for tracked corrosion patches across consecutive frames, allowing the navigation controller to adjust the vehicle heading as tracked defects moved through the field of view.

---

# Repository Contents

| File | Description |
|------|-------------|
| `Final_Code.py` | Pipeline segmentation and navigation controller implementing HSV thresholding, contour and orientation estimation, Hough line detection, and `pyautogui` control for Minecraft |
| `Demo_Sample.mp4` | Demonstration of a successful navigation run |
| `Trial_Error.mp4` | Demonstration of a run exhibiting one of the identified failure cases |
| `AUV simulation for underwater pipeline inspection.pdf` | Project presentation containing the problem statement, methodology, dataset, and results |
| `NUS Report Work.pdf` | Internship report |

YOLOv8s training and DeepSORT tracking were performed separately using Roboflow-hosted training and external notebooks that are not included in this repository. This repository contains only the navigation controller.

---

# Running Final_Code

Install the required dependencies:

```bash
pip install opencv-python pyautogui numpy
```

Open the Minecraft world containing the constructed pipeline.

Position the Minecraft window so that the pipeline lies within the screen capture region hardcoded in the script. The default capture region is:

- Top-left corner: `(100, 100)`
- Width: `700 px`
- Height: `500 px`

Modify the `region` variable near the beginning of `Final_Code.py` if your display resolution or window placement differs.

Run the controller:

```bash
python Final_Code.py
```

Switch to the Minecraft window during the five-second startup delay.

Three OpenCV windows will appear:

- HSV binary mask
- Hough line overlay
- Main view showing the fitted bounding box and pipeline orientation

Press **q** in any window to terminate the program.

---

# Platform Notes

`pyautogui` requires operating system permissions to capture the screen and send keyboard and mouse input.

- **macOS:** Grant Accessibility and Screen Recording permissions.
- **Linux:** An X11 session is required. Wayland is not currently supported.

---

# Limitations

- The interface between Python and Minecraft was occasionally unstable because of compatibility issues between the game and the automation script.
- Some data was lost during frame extraction and annotation.
- The HSV thresholds and screen capture region are tuned for a specific texture pack and window configuration. Different environments require recalibration of these parameters.

---

# Team

This project was completed during a research internship at the **National University of Singapore (NUS)** in June 2025 under the guidance of **Dr. Amirhassan Monajemi**.

- Roland Singh
- Ayush Upadhyay
- Khushi J. H.
- Kirat Singh
- Tarun Malathi Raja
