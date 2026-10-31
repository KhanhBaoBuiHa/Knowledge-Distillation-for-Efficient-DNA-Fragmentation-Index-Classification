# Knowledge Distillation for Efficient DNA Fragmentation Index Classification

Undergraduate thesis project (Mathematics & Computer Science, Data Science major, HCMUS): a lightweight deep-learning pipeline that classifies sperm images from SCD (Sperm Chromatin Dispersion) microscopy, using **knowledge distillation** to compress large teacher models into small students that are cheap to run on CPU.

> 🚧 **Work in progress.** The accompanying research paper is still being written. Code, results and documentation in this repository may change. Final numbers, citation details and a preprint link will be added once the paper is ready.

---

## Overview

The SCD test estimates the DNA Fragmentation Index (DFI) from the size of the halo around each sperm cell. Doing this by hand is slow and subjective. This project:

1. **Benchmarks** several strong image classifiers (teachers) on two tasks.
2. **Distills** the best teachers into compact students (MobileNetV3, EfficientNet-Lite0) with logit-based KD, ordinal KD and feature-based **Relaxed KD (ReKD)**.
3. **Compares** teachers, students trained from scratch (baseline) and distilled students on accuracy, model size and CPU latency.

Dataset: Saadat et al., *Scientific Data* (2026), hosted on Figshare (article ID `30120811`). The notebooks download it automatically.

## Tasks

| Task | Description | Classes | Test / Val images |
|------|-------------|---------|-------------------|
| **A** | Binary classification | Sperm / Non-sperm | 98 / 98 |
| **B** | Halo-size classification (proxy for DNA fragmentation) | Large halo, Medium halo, Small halo, Without halo | 61 / 61 |

## Pipeline

See `pipeline_v2.jfif` (current) and `pipeline.jfif` (earlier version) for the workflow diagram.

## Repository structure

```
.
├── TaskA_Binary_Classification.ipynb         # Task A: teacher benchmarking
├── TaskB_Multiclass_Classification.ipynb     # Task B: teacher benchmarking
├── Relaxed_KD_V3_Task_A.ipynb                # Distillation experiments, Task A
├── Relaxed_KD_V3_Task_B.json                 # Distillation experiments, Task B
├── Evaluation.ipynb / Evaluation_v2.ipynb    # Test-set evaluation + Grad-CAM
├── Final of ALL_TASKS_evaluation_results.xlsx  # Teacher test-set results
├── distill_kq.xlsx                           # Distillation results, student sizes, configs
├── Các bài báo.xlsx                          # Literature review notes
└── pipeline.jfif / pipeline_v2.jfif          # Pipeline diagrams
```

## Method

**Teachers (benchmarked):** ResNet50, ConvNeXt-Base, EfficientNetV2-M, ViT-Base, Swin-Base, DINOv2.
**Students:** MobileNetV3-Small, MobileNetV3-Large, EfficientNet-Lite0.

**Training variants**

| Variant | Description |
|---------|-------------|
| `baseline` | Student trained with cross-entropy only |
| `kd` | Logit KD from a teacher ensemble: `(1−λ)·CE + λ·T²·KL` |
| `kd_ord` | Logit KD with ordinal soft labels (Task B only; Large < Medium < Small < Without halo) |
| `rekd` | Feature-based Relaxed KD (VAE-style encoder/decoder on teacher features) from a single teacher |
| `kd_rekd` | Logit KD + ReKD combined |

ReKD follows Qu et al. (IJCV 2026). It is **re-implemented from the paper's description, not the official code**; deviations are documented in the `Config` sheet of `distill_kq.xlsx`.

**Training setup:** 2-stage fine-tuning (head-only, then full), AdamW (weight decay 1e-4), early stopping on validation macro-F1, flips / ±180° rotation / light color jitter, seeds 42, 43, 44. KD hyperparameters: T = 4, λ = 0.5, α = 0.05, β = 10. The test split is not used during distillation.

## Results

> ⚠️ Preliminary. Evaluation sets are small (98 images for Task A, 61 for Task B), so one image is worth roughly 1–1.6 points of F1 and small differences may be noise.

### 1. Teacher benchmark (test set)

**Task A: Sperm vs. Non-sperm**

| Model | Accuracy | Macro-F1 |
|-------|:--------:|:--------:|
| ResNet50 | **0.9286** | **0.9286** |
| ConvNeXt-Base | 0.9082 | 0.9079 |

**Task B: Halo classification**

| Model | Accuracy | Macro-F1 |
|-------|:--------:|:--------:|
| Swin-Base | **0.9016** | **0.9017** |
| ConvNeXt-Base | 0.8852 | 0.8821 |
| ViT-Base | 0.8689 | 0.8641 |
| DINOv2 | 0.8197 | 0.8030 |
| EfficientNetV2-M | 0.7213 | 0.7138 |
| ResNet50 | 0.6066 | 0.5826 |

### 2. Distillation (validation set, macro-F1, mean ± std over 3 seeds)

Tags: `L-` = logit teacher, `F-` = feature teacher; `cn` = ConvNeXt, `sw` = Swin, `vt` = ViT, `r5` = ResNet50. Δ = difference vs. the baseline of the same student.

**Task B: Halo classification** (best teacher, Swin-Base: val macro-F1 0.9146)

| Student | Config | Val macro-F1 | Δ vs baseline | Ordinal MAE ↓ |
|---------|--------|:------------:|:-------------:|:-------------:|
| MobileNetV3-Small | baseline | 0.8629 ± 0.0497 | n/a | 0.1421 |
| MobileNetV3-Small | kd `[L-sw]` | 0.8638 ± 0.0143 | +0.0009 | 0.1366 |
| MobileNetV3-Small | kd_ord `[L-sw]` | 0.8441 ± 0.0209 | −0.0188 | 0.1585 |
| MobileNetV3-Small | rekd `[F-sw]` | 0.8876 ± 0.0267 | +0.0247 | 0.1202 |
| MobileNetV3-Small | **kd_rekd `[L-sw__F-sw]`** | **0.8980 ± 0.0128** | **+0.0351** | **0.1038** |
| MobileNetV3-Large | baseline | 0.8194 ± 0.0288 | n/a | 0.2131 |
| MobileNetV3-Large | kd `[L-cn-sw-vt]` | 0.8171 ± 0.0751 | −0.0023 | 0.1967 |
| MobileNetV3-Large | rekd `[F-sw]` | 0.8104 ± 0.0125 | −0.0090 | 0.2240 |
| MobileNetV3-Large | kd_rekd `[L-sw-vt__F-sw]` | 0.8322 ± 0.0179 | +0.0128 | 0.1803 |
| EfficientNet-Lite0 (1 seed) | baseline / kd / kd_ord | 0.7553 / 0.7263 / 0.6943 | n/a | n/a |

**Task A: Sperm vs. Non-sperm** (teachers: ConvNeXt-Base 0.9898, ResNet50 0.9592, ensemble 0.9898)

| Student | Config | Val macro-F1 | Δ vs baseline |
|---------|--------|:------------:|:-------------:|
| MobileNetV3-Large | baseline | 0.9898 ± 0.0000 | n/a |
| MobileNetV3-Large | kd `[L-cn-r5]` | 0.8806 ± 0.0822 | −0.1092 |
| MobileNetV3-Large | kd_rekd `[L-cn-r5__F-cn]` | 0.9626 ± 0.0059 | −0.0272 |
| MobileNetV3-Large | rekd `[F-cn]` | 0.9932 ± 0.0118 | +0.0034 |
| EfficientNet-Lite0 (1 seed) | baseline / kd | 0.9796 / 0.9284 | n/a |

### 3. Student size and CPU latency

| Student | Params (M) | Size FP32 (MB) | CPU latency (ms) |
|---------|:----------:|:--------------:|:----------------:|
| MobileNetV3-Small | 1.52 | 6.09 | 9.7–10.0 |
| MobileNetV3-Large | 4.21 | 16.8 | 19.4–22.7 |
| EfficientNet-Lite0 | 3.37 | 13.5 | 28.1–44.3 |

### Takeaways (so far)

- **Task B:** the combined `kd_rekd` setup gives the best student: MobileNetV3-Small (1.5M params, ~6 MB, ~10 ms on CPU) reaches 0.898 val macro-F1, within about 2 points of the Swin-Base teacher (0.915) and +3.5 points over its own baseline. Plain logit KD and ordinal KD did not clearly help.
- **Task A:** the baseline student already matches the teachers on validation, so there is little headroom. ReKD gives +0.003, which is within noise, and logit KD actually hurts.
- **Caveats:** distilled students have only been evaluated on the validation set, and the teacher ranking differs between validation and test (e.g. ConvNeXt vs. ResNet50 in Task A), which again points to small-sample noise.

## Getting started

The notebooks were developed on Google Colab with PyTorch and `timm`.

```bash
git clone https://github.com/KhanhBaoBuiHa/Knowledge-Distillation-for-Efficient-DNA-Fragmentation-Index-Classification.git
cd Knowledge-Distillation-for-Efficient-DNA-Fragmentation-Index-Classification
pip install torch torchvision timm==1.0.11 scikit-learn pandas numpy matplotlib seaborn grad-cam openpyxl jupyter
```

1. Open the notebooks in Colab (or Jupyter) and update the Google Drive / data path variables at the top; the dataset is downloaded from Figshare automatically.
2. Run `TaskA_Binary_Classification.ipynb` and `TaskB_Multiclass_Classification.ipynb` to train and benchmark the teachers.
3. Run the `Relaxed_KD_V3_*` notebooks to distill the teachers into students.
4. Run `Evaluation_v2.ipynb` for test-set metrics, confusion matrices and Grad-CAM.

## Status & roadmap

- [x] Teacher benchmarking (Task A, Task B)
- [x] Logit KD, ordinal KD and ReKD experiments on the validation set
- [x] Teacher test-set evaluation and Grad-CAM
- [ ] Test-set evaluation of the distilled students
- [ ] Re-check Task A baseline variance (3 seeds currently give identical scores)
- [ ] More seeds / cross-validation to reduce small-sample noise
- [ ] `requirements.txt` and cleaner training scripts
- [ ] Paper submission and preprint link

## Citation

The paper is in preparation. Until it is available, please cite the repository:

```bibtex
@misc{bui2026kd_dfi,
  author = {Bùi Hà Bảo Khanh},
  title  = {Knowledge Distillation for Efficient DNA Fragmentation Index Classification},
  year   = {2026},
  howpublished = {\url{https://github.com/KhanhBaoBuiHa/Knowledge-Distillation-for-Efficient-DNA-Fragmentation-Index-Classification}},
  note   = {Work in progress; paper in preparation}
}
```

Please also cite the dataset paper (Saadat et al., *Scientific Data*, 2026) and the ReKD paper (Qu et al., *IJCV*, 2026).

## Author

**Bùi Hà Bảo Khanh**, Mathematics & Computer Science (Data Science), University of Science, VNU-HCM
Advisor: *[advisor name]*

## License

*[Add a license, e.g. MIT, or state "All rights reserved" until the paper is published.]*
