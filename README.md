# Dam Structural Health Monitoring
## Deep Learning-Driven Crack Quantification and Remaining Useful Life Prediction


> **Paper:** *Deep Learning-Driven Crack Quantification and Remaining Useful Life Prediction for Dam Structural Health Monitoring*
> Vineet Santosh Bhoir — IIT Kharagpur, Dept. of Industrial and Systems Engineering

---

## Overview

This project presents an end-to-end pipeline for automated structural health monitoring of concrete dams. A **ResNet50 CNN** predicts crack width directly from field photographs, which are then fed into a **Linear Elastic Fracture Mechanics (LEFM)** engine to compute the stress intensity factor K₁ and estimate **Remaining Useful Life (RUL)** via Paris' Law integration.

### Key Results
| Metric | Value |
|--------|-------|
| Mean Width MAE | **0.48 mm** |
| Max Width MAE | **1.27 mm** |
| Mean Width RMSE | 0.72 mm |
| Max Width RMSE | 1.63 mm |
| Training Epochs | 80 |
| Dataset Size (after augmentation) | 450 images |

---

## Pipeline

```
Crack Image
    ↓  ResNet50 CNN
Crack Width Prediction  (mean_width_mm, max_width_mm)
    ↓  + Mask CV (length, orientation)
LEFM Computation  →  K₁ = Y · σₙ · √(πa)
    ↓  Paris' Law Integration
RUL Estimate  +  Risk Classification
```

---

## Dataset

This project uses the **Virginia Tech Crack Quantification ROIs Dataset**:
- 15 physical concrete crack specimens
- 3 standoff distances: 0.5 m, 0.75 m, 1.0 m → **45 images**
- 5 manually measured ROI widths per image (ruler-calibrated, mm)
- Binary segmentation masks provided

**Download:** [Virginia Tech Data Repository](https://data.lib.vt.edu/articles/dataset/Crack_Quantification_ROIs_Dataset/29361029)

After downloading, the folder structure should look like:
```
Dataset/
├── Cropped/
│   ├── 0.5/Sample_1/Sample_1_0.5_cropped.JPG
│   ├── 0.75/...
│   └── 1.0/...
├── Ground_Truth_Masks/
│   ├── 0.5/Sample_1/Sample_1_0.5_cropped.jpg
│   └── ...
└── ROI_Measurements.csv
```

---

## Quickstart (Kaggle)

The notebook is designed to run on **Kaggle** with GPU enabled.

1. Upload the dataset to Kaggle Datasets
2. Add it as input to your notebook
3. Open `Dam_SHM_Final.ipynb`
4. In **Section 1**, set:
```python
DATASET_ROOT = Path('/kaggle/input/your-dataset-name/Dataset')
```
5. Run all cells top to bottom

> **Note:** Set `NUM_WORKERS = 0` if running locally on Windows.

---

## Requirements

```
torch>=2.0.0
torchvision>=0.15.0
opencv-python-headless>=4.7.0
Pillow>=9.0.0
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
scikit-learn>=1.2.0
```

Install with:
```bash
pip install -r requirements.txt
```

---

## Notebook Structure

| Section | Description |
|---------|-------------|
| 0 · Imports | All dependencies |
| 1 · Configuration | Paths, hyperparameters, dam physics constants |
| 2 · Build labels.csv | Aggregates ROI widths + extracts mask geometry |
| 3 · Augmentation | 45 → 450 images via 9 transform strategies |
| 4 · Split & Stats | Stratified train/val/test split, label normalisation |
| 5 · Dataset & Loaders | PyTorch Dataset class with online augmentation |
| 6 · CNN Model | ResNet50 + shared layer + 2 regression heads |
| 7 · Training | AdamW, CosineAnnealingWarmRestarts, mixed precision |
| 8 · Inference & Evaluation | predict(), test metrics, scatter plots |
| 9 · RUL Engine | LEFM + Paris' Law numerical integration |
| 10 · Dashboard | Health index visualisation across all images |
| 11 · Single-image Inspection | Report card for any new crack photo |

---

## Physics Model

**Hydrostatic pressure:**

$$p(y) = \rho g y$$

**Normal stress on crack plane:**

$$\sigma_n = \frac{p(y)}{t} \cos^2(\theta)$$

**Mode-I Stress Intensity Factor:**

$$K_I = Y \sigma_n \sqrt{\pi a}$$

**Paris' Law crack growth:**

$$\frac{da}{dt} = C (K_I)^m$$

**RUL:**

$$\text{RUL} = \sum \Delta t \quad \text{until} \quad K_I \geq K_{IC}$$

| Parameter | Value |
|-----------|-------|
| Dam height H | 100 m |
| Wall thickness t | 12 m |
| Fracture toughness K₁c | 1.0 MPa√m |
| Geometry factor Y | 1.1 |
| Paris constant C | 10⁻¹² |
| Paris exponent m | 2.5 |

---

## Risk Classification

| K₁ / K₁c | Risk Level |
|-----------|------------|
| ≥ 1.0 | 🔴 CRITICAL — Immediate action |
| 0.8 – 1.0 | 🟠 HIGH — Urgent inspection |
| 0.6 – 0.8 | 🟡 MODERATE — Close monitoring |
| 0.4 – 0.6 | 🟢 ELEVATED — Routine monitoring |
| < 0.4 | ✅ LOW — Normal operation |

---

## Repository Structure

```
dam-shm-crack-rul/
├── README.md
├── requirements.txt
├── notebook/
│   └── Dam_SHM_Final.ipynb
├── paper/
│   └── APARM_Paper_v3.docx
└── assets/
    └── pipeline.jpg
```

---

## Citation

If you use this work, please cite:

```bibtex
@inproceedings{bhoir2025damshm,
  title     = {Deep Learning-Driven Crack Quantification and Remaining Useful Life
               Prediction for Dam Structural Health Monitoring},
  author    = {Bhoir, Vineet Santosh},
  booktitle = {Proceedings of APARM 2026},
  year      = {2025},
  institution = {IIT Kharagpur}
}
```

---

## License

This project is released under the **MIT License**.
The dataset is released under **CC BY 4.0** by Virginia Tech — please cite the original dataset when using it.

---

## Contact

**Vineet Santosh Bhoir**
Department of Industrial and Systems Engineering, IIT Kharagpur
vineetbhoir@kgpian.iitkgp.ac.in
