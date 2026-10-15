# Endoscope Semantic Segmentation for Computer-Assisted Surgery

Semantic segmentation of laparoscopic/endoscopic images using a **U-Net** model to label tissues, instruments, blood vessels, and other structures. The goal is pixel-accurate scene understanding to support computer-assisted surgery.

> **Dataset note:** The dataset is not included in this repo due to size. You can download it from Kaggle: **[CholecSeg8K](https://www.kaggle.com/datasets/newslab/cholecseg8k)**. Place it under `data/` as shown below.

---

## Highlights
- **Task:** Multi-class semantic segmentation (**13 classes**) on endoscopic frames  
- **Model:** U-Net (encoder–decoder with skip connections)  
- **Masks:** Expert annotation → **color mask** (visualization) + **watershed mask** (grayscale class IDs for training)  
- **Split:** Train / Val / Test = **60 / 20 / 20**  
- **Results (test set):** Pixel Acc. **98.94%**, IoU **50.19%**, Dice **51.67%**

---

## Table of Contents
- [Dataset](#dataset)
- [Mask Overview](#mask-overview)
- [Class Information](#class-information)
- [Model](#model)
- [Results](#results)
- [Setup](#setup)
- [Quickstart](#quickstart)
- [Training & Evaluation](#training--evaluation)
- [Cite & Acknowledgements](#cite--acknowledgements)
- [Contact](#contact)

---

## Dataset

Each video is split into segments of 80 frames. For each frame you get:
- **Raw image frame**
- **Annotation tool mask** (hand-drawn)
- **Color mask** (RGB colors per class for visualization)
- **Watershed mask** (grayscale class IDs for training)

<p align="center">
  <img src="Images/Fig1.png" alt="Dataset examples: raw frames and color masks" width="850">
  <img src="Images/Fig2.png" alt="Dataset examples: raw frames and color masks" width="850">
  <img src="Images/Fig3.png" alt="Dataset examples: raw frames and color masks" width="850">
</p>

---

## Mask Overview

The dataset provides three mask variants per frame. Use the **watershed** masks (integer IDs) for training.

<table>
  <thead>
    <tr>
      <th>Mask Name</th>
      <th>Description</th>
      <th>Image</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Original Image Frame</strong></td>
      <td>This is the raw endoscopic image captured during the surgery.</td>
      <td><img src="Images/frame_100_endo.png" alt="Mask overview composite (includes raw, annotation, color, watershed)" width="320"></td>
    </tr>
    <tr>
      <td><strong>Annotation Tool Mask</strong></td>
      <td>
        <ul>
          <li>Original hand-drawn mask created during annotation.</li>
          <li>Pixel-level expert labels; base for generating other masks.</li>
        </ul>
      </td>
     <td><img src="Images/frame_100_endo_mask.png" alt="Mask overview composite (includes raw, annotation, color, watershed)" width="320"></td>
    </tr>
    <tr>
      <td><strong>Color Mask</strong></td>
      <td>
        <ul>
          <li>Derived from the annotation mask.</li>
          <li>Assigns a unique <em>color</em> to each class for easy visual inspection.</li>
        </ul>
      </td>
      <td><img src="Images/frame_100_endo_color_mask.png" alt="Mask overview composite (includes raw, annotation, color, watershed)" width="320"></td>
    </tr>
    <tr>
      <td><strong>Watershed Mask</strong></td>
      <td>
        <ul>
          <li>Also derived from the annotation mask.</li>
          <li>Assigns a uniform pixel value (same across R,G,B) to represent class IDs.</li>
          <li>Use these integer IDs for training (no one-hot in files).</li>
        </ul>
      </td>
       <td><img src="Images/frame_100_endo_watershed_mask.png" alt="Mask overview composite (includes raw, annotation, color, watershed)" width="320"></td>
    </tr>
  </tbody>
</table>

> The image cell shows a compact composite that contains the example rows (raw, annotation, color, watershed).

---

## Class Information

Below are the **13 classes** used in the watershed masks. (IDs are the pixel values in the grayscale masks; the RGB triplet is the same value repeated across channels.)

| Class ID | Class Name             | RGB Hexcode |
|:-------:|------------------------|:-----------:|
| 0       | Black Background       | `#505050`   |
| 1       | Abdominal Wall         | `#111111`   |
| 2       | Liver                  | `#212121`   |
| 3       | Gastrointestinal Tract | `#131313`   |
| 4       | Fat                    | `#121212`   |
| 5       | Grasper                | `#313131`   |
| 6       | Connective Tissue      | `#232323`   |
| 7       | Blood                  | `#242424`   |
| 8       | Cystic Duct            | `#252525`   |
| 9       | L-hook Electrocautery  | `#323232`   |
| 10      | Gallbladder            | `#222222`   |
| 11      | Hepatic Vein           | `#333333`   |
| 12      | Liver Ligament         | `#050505`   |


---

## Model
- **Architecture:** U-Net with skip connections (strong baseline for biomedical segmentation)
- **Loss:** `SparseCategoricalCrossentropy` (works directly with integer masks)
- **Optimizer:** Adam
- **Callback:** `EarlyStopping(patience=5, restore_best_weights=True)`
- **Metrics during training:** pixel accuracy  
- **Post-training:** compute **mean IoU** and **Dice (F1)** on the **test** set

<p align="center">
  <img src="Images/U-Net-convolutional-neural-network-architecture.png" alt="Class information reference table (with example hex codes)" width="700">
</p>

---

## Results
From the current configuration (see notebook and `reports/`):

| Metric                 | Score   |
|------------------------|--------:|
| Accuracy (Keras)       | 0.9638  |
| Loss                   | 0.2697  |
| **Pixel Accuracy**     | **96.39%** |
| **IoU (mean)**         | **72.23%** |
| **Dice (F1, mean)**    | **76.89%** |

> These are baseline results. See the Roadmap for improvement ideas.

---

## Setup
> Python **3.10+** recommended.

**Conda (recommended):**
```bash
conda create -n endo-seg python=3.10 -y
conda activate endo-seg
pip install -U pip
pip install tensorflow matplotlib numpy opencv-python scikit-image albumentations tqdm jupyter tabulate
```

**Virtualenv
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
source .venv/bin/activate
pip install -U pip
pip install tensorflow matplotlib numpy opencv-python scikit-image albumentations tqdm jupyter tabulate
```
---

## Quickstart

- Download the dataset from Kaggle and place it under `data/` following the structure above.
- Open the notebook:

    jupyter notebook notebooks/medical_image_segm.ipynb

- Run all cells to:
  - load & preprocess data (color vs. watershed masks)
  - train U-Net with early stopping
  - evaluate on the test set
  - visualize predictions (overlays, color masks)

## Training & Evaluation

**Key settings (mirrored in the notebook):**
- Split: **60/20/20** (train/val/test)
- Loss: `SparseCategoricalCrossentropy` (integer masks)
- Optimizer: `Adam`
- Callback: `EarlyStopping(patience=5, restore_best_weights=True)`
- Metric during training: pixel accuracy
- Post-training: compute **mean IoU** and **Dice** on the test set

> **Important:** With watershed (grayscale) masks, read labels as integer IDs (no one-hot). Use a **softmax** output with `from_logits=False`.

## Cite & Acknowledgements

- Slides/figures: see `reports/Endoscope Semantic Segmentation.pdf`.
- Dataset: **CholecSeg8K** — https://www.kaggle.com/datasets/newslab/cholecseg8k

## Contact

Mariana-Ionela Muntian  
**LinkedIn:** https://linkedin.com/in/ionelamuntian  
**GitHub:** https://github.com/ionelaMuntian
