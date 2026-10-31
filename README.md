# Knowledge Distillation for Efficient DNA Fragmentation Index Classification

Undergraduate thesis project (Mathematics & Computer Science, Data Science major, HCMUS): a lightweight deep-learning pipeline that classifies sperm DNA fragmentation (halo) patterns from SCD microscopy images, using **knowledge distillation** to keep accuracy close to large models at a fraction of the compute.

> 🚧 **Work in progress.** The accompanying research paper is still being written, and the code, results and documentation in this repository may change. Final numbers, citation details and a preprint link will be added once the paper is ready.

---

## Overview

The Sperm Chromatin Dispersion (SCD) test estimates the DNA Fragmentation Index (DFI) by looking at the halo around each sperm cell. Doing this manually is slow and subjective. This project studies how to automate it with models that are small enough to be practical:

1. Benchmark several strong classification models (teachers) on the task.
2. Distill the best-performing teacher into a compact student with **relaxed knowledge distillation**.
3. Compare teachers, students trained from scratch, and distilled students on accuracy and efficiency.

The data comes from the Saadat et al. sperm-image dataset (*Scientific Data*, 2026).

## Tasks

| Task | Description | Notebook |
|------|-------------|----------|
| **Task A** | Binary classification: sperm vs. non-sperm | `TaskA_Binary_Classification.ipynb` |
| **Task B** | Multi-class classification of the halo / DNA fragmentation level | `TaskB_Multiclass_Classification.ipynb` |

## Pipeline

See `pipeline.jfif` (v1) and `pipeline_v2.jfif` (current version) for the overall workflow diagram.

## Repository structure

```
.
├── TaskA_Binary_Classification.ipynb       # Task A: baseline / teacher benchmarking
├── TaskB_Multiclass_Classification.ipynb   # Task B: baseline / teacher benchmarking
├── Relaxed_KD_V3_Task_A.ipynb              # Relaxed knowledge distillation, Task A
├── Relaxed_KD_V3_Task_B.json               # Relaxed knowledge distillation, Task B
├── Evaluation.ipynb                        # Evaluation of all models (v1)
├── Evaluation_v2.ipynb                     # Evaluation of all models (v2)
├── Final of ALL_TASKS_evaluation_results.xlsx   # Consolidated results, all tasks
├── distill_kq.xlsx                         # Distillation experiment results
├── Các bài báo.xlsx                        # Literature review notes
├── pipeline.jfif / pipeline_v2.jfif        # Pipeline diagrams
```

## Getting started

The notebooks were developed in Jupyter / Google Colab with PyTorch.

```bash
git clone https://github.com/KhanhBaoBuiHa/Knowledge-Distillation-for-Efficient-DNA-Fragmentation-Index-Classification.git
cd Knowledge-Distillation-for-Efficient-DNA-Fragmentation-Index-Classification
pip install torch torchvision scikit-learn pandas numpy matplotlib jupyter openpyxl
```

1. Download the dataset from the original publication and update the data path variables at the top of each notebook.
2. Run the Task A / Task B notebooks to train and benchmark the teacher models.
3. Run the `Relaxed_KD_V3_*` notebooks to distill the best teacher into the student.
4. Run `Evaluation_v2.ipynb` to reproduce the comparison tables.

*(Exact package versions and a `requirements.txt` will be added.)*

## Results

Final results are summarized in `Final of ALL_TASKS_evaluation_results.xlsx` and `distill_kq.xlsx`. A results table (teacher vs. student: accuracy, macro-F1, parameters, inference time) will be added here once the experiments are finalized for the paper.

## Status & roadmap

- [x] Teacher benchmarking for Task A and Task B
- [x] Relaxed knowledge distillation experiments
- [x] Evaluation notebooks and result tables
- [ ] Final results table in this README
- [ ] Cleaner, reproducible training scripts and `requirements.txt`
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

Please also cite the original dataset paper (Saadat et al., *Scientific Data*, 2026).

## Author

**Bùi Hà Bảo Khanh** — Mathematics & Computer Science (Data Science), University of Science, VNU-HCM
Advisor: *[advisor name]*

## License

*[Add a license, e.g. MIT, or state "All rights reserved" until the paper is published.]*
