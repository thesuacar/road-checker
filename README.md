# 5ARE0 Assignment 1 — Assessing Bicycle Lane Quality Using Smartphone Sensor Data

**Course:** 5ARE0 — Data Analysis & Learning Methods  
**Academic year:** 2026–2027  
**Group:** 5
## Environment Dependencies

```
Python: 3.12.14
NumPy: 2.5.2
Pandas: 3.0.5
Matplotlib: 3.11.1
SciPy: 1.18.1
scikit-learn: 1.9.0
```

## Project overview

This project investigates the classification of bicycle-lane surface quality as **Smooth** or **Bumpy** using smartphone sensor data.

The analysis follows the CRISP-DM framework and includes:

- data loading and quality screening;
- signal preprocessing;
- exploratory data analysis;
- physics-informed feature engineering;
- feature selection;
- supervised learning;
- unsupervised learning;
- model evaluation and comparison;
- deployment on an independent external dataset.

A detailed description of the dataset, methodology, modelling decisions and results is provided in `notebook.ipynb` and `report.pdf`.

---

## Submission structure

```text
.
├── README.md
├── notebook.ipynb
├── report.pdf
└── photos-videos/
└── data/
    ├── checkpoints/
    ├── deployment_data/
    ├── downloads/
    ├── extracted/
    ├── model_handoff/
    └── model_results/