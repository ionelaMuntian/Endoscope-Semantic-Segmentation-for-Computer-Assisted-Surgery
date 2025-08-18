# Endoscope Semantic Segmentation

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
- [Project Structure](#project-structure)
- [Dataset](#dataset)
- [Model](#model)
- [Results](#results)
- [Setup](#setup)
- [Quickstart](#quickstart)
- [Training & Evaluation](#training--evaluation)
- [Notes & Roadmap](#notes--roadmap)
- [Cite & Acknowledgements](#cite--acknowledgements)
- [Contact](#contact)

---

## Project Structure
.
├─ notebooks/
│ └─ medical_image_segm.ipynb # end-to-end workflow
├─ reports/
│ └─ Endoscope Semantic Segmentation.pdf
├─ models/ # saved weights (gitignored)
├─ data/ # dataset (gitignored)
├─ outputs/ # predictions & figures (gitignored)
└─ README.md

---

## Dataset
**Source:** *CholecSeg8K* (cholecystectomy scenes). Each video is split into segments of 80 frames. For each frame you get four images:
- **Raw frame**
- **Annotation mask (color)** — for visualization
- **Watershed mask (grayscale IDs)** — **use this for training**
- *(Sometimes)* additional helper masks

Every frame is pixel-wise annotated for **13 classes** (e.g., abdominal wall, liver, fat, grasper, blood, cystic duct, gallbladder, hepatic vein).

**Expected local layout (example):**
data/
images/
train/.png
val/.png
test/.png
masks_id/ # grayscale, integer class IDs (training)
train/.png
val/.png
test/.png
masks_color/ # optional, visualization
train/.png
val/.png
test/*.png

⚠️ Respect the dataset’s license/terms. Keep large data out of version control.

---

## Model
- **Architecture:** U-Net with skip connections (strong baseline for biomedical segmentation)
- **Loss:** `SparseCategoricalCrossentropy` (works directly with integer masks)
- **Optimizer:** Adam
- **Callback:** `EarlyStopping(patience=5, restore_best_weights=True)`
- **Metrics during training:** pixel accuracy  
- **Post-training:** compute **mean IoU** and **Dice (F1)** on the **test** set

---

## Results
From the current configuration (see notebook and `reports/`):

| Metric                 | Score   |
|------------------------|--------:|
| Accuracy (Keras)       | 0.9899  |
| Loss                   | 0.0414  |
| **Pixel Accuracy**     | **98.94%** |
| **IoU (mean)**         | **50.19%** |
| **Dice (F1, mean)**    | **51.67%** |

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

**Virtualenv
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
