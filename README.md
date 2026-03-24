# ecoli-dna-promoter-automl

A Python project that detects E. Coli promoter sequences from raw DNA strings using AutoML.

What makes it different from a standard classification project is that the input is not numbers or images — it is raw DNA text (strings of A, C, G, T characters). The project converts those strings into meaningful numerical features using biology-informed techniques, then uses AutoML tools to automatically find and tune the best model. I built this in Google Colab using the UCI E. Coli Promoter Gene Sequences dataset.

---

## What it does

You give it a 57-character DNA sequence. The pipeline engineers features from the sequence, scales them, runs them through the best model found by AutoML, and tells you whether the sequence is a promoter or not — along with a confidence score and an explanation of which features drove the prediction.

---

## How it works

The pipeline runs through these steps:

1. **Setup** — detects Colab vs local, installs libraries, mounts Google Drive, sets seeds
2. **Config** — all parameters in one place, nothing hardcoded anywhere else
3. **Download** — fetches UCI dataset with fallback, creates artificial imbalance to demo SMOTE and class_weight
4. **EDA** — GC content analysis, nucleotide frequencies, position-wise heatmaps, k-mer enrichment charts
5. **Feature Engineering** — converts raw DNA strings into 86 numerical features using k-mer counts and biological statistics
6. **Baseline** — trains Logistic Regression and Random Forest manually before AutoML
7. **LazyPredict** — compares 20+ sklearn models automatically in one call
8. **FLAML** — finds and tunes the best model within a 120 second time budget
9. **Comparison** — builds a master table of all approaches and selects the winner by AUC
10. **Evaluation** — confusion matrix, ROC curve, precision-recall curve, optimal threshold, 5-fold CV
11. **SHAP** — explains which features drove each prediction, maps them to known E. Coli promoter motifs
12. **Save** — versioned model checkpoint with metadata, scaler, feature names and threshold bundled together
13. **Prediction function** — reusable `predict_ecoli()` with full input validation and error handling
14. **Gradio UI** — web interface with confidence score, top features, sequence stats, and example sequences

---

## Feature engineering

This is the most unique part of the project. Standard ML projects use ready-made numerical inputs. Here the raw input is a DNA string like `TTGACAATTAATCATCGAAC...` which no model can process directly. The pipeline builds 86 features from each sequence:

| Feature Type | What it captures | Count |
|---|---|---|
| 2-mer frequencies | How often each 2-character substring appears (AA, AC, AG...) | 16 |
| 3-mer frequencies | How often each 3-character substring appears (AAA, AAC...) | 64 |
| GC content | Percentage of G and C bases — biological signal | 1 |
| AT content | Percentage of A and T bases | 1 |
| Nucleotide frequencies | Individual A%, C%, G%, T% per sequence | 4 |

---

## AutoML comparison based on:

5-Fold Cross Validation Mean AUC confirms the model is robust and not just lucky on one test split.

---

## Imbalance handling

The original dataset is perfectly balanced (50/50). To make the project more realistic and demonstrate imbalance techniques, 30 Non-Promoter samples were removed artificially to create a 70/30 split. The pipeline then demonstrates:

- `class_weight="balanced"` in baseline models
- SMOTE available as an option for severe imbalance
- AUC used as primary metric instead of accuracy (accuracy is misleading on imbalanced data)

---

## SHAP explainability

After prediction, SHAP values explain why the model made each decision. The top features are mapped back to known E. Coli biology:

- **3mer_TAT, 3mer_ATA, 3mer_AAT** — substrings of the TATAAT -10 box (Pribnow box), the most conserved promoter element in E. Coli
- **3mer_TTG, 3mer_TGA** — substrings of the TTGACA -35 box, the upstream recognition element
- **at_pct, gc_pct** — base composition signals, promoters tend to be AT-rich

---

## Tech stack

- Python
- scikit-learn
- LazyPredict
- FLAML
- SHAP
- pandas, numpy
- matplotlib, seaborn
- Gradio
- Google Colab

---

## How to run it

**Option 1 — Google Colab (easiest)**

- Upload the step files to Colab or paste into cells
- Run steps 0 to 13 in order, each step in a separate cell
- The last step launches a Gradio web app with a public share link automatically

**Option 2 — Run locally**

Make sure Python is installed then run:

```
pip install -r requirements.txt
```

Then run each step file in order from step00 to step13.

---

## Requirements

All dependencies are in `requirements.txt`. Main ones are:

```
numpy
pandas
scikit-learn
lazypredict
flaml
shap
gradio
matplotlib
seaborn
biopython
ucimlrepo
joblib
```

---

## Project structure

```
ecoli-dna-promoter-automl/
│
├── ecoli_dna_automl_complete.py  
├── requirements.txt
├── README.md
└── .gitignore
```

---

## What I learned building this

The biggest challenge was that the raw input is a DNA string, not numbers. Every standard ML tutorial assumes your features are already numerical. Here I had to think about what biological properties of a DNA sequence actually matter for promoter detection — GC content, k-mer patterns, positional conservation — and build those features manually before AutoML could even start.

The second lesson was about data leakage. The StandardScaler must be fit only on training data. Fitting it on the full dataset before splitting would let test data statistics influence the training process, making evaluation results look better than they really are.

The third lesson was about model confidence. Our model gives ~79% confidence on predictions. With only 76 samples this is actually appropriate — a model showing 99% confidence on such a small dataset would be a red flag for overfitting. AUC = 1.0 means the model ranks sequences correctly; the moderate confidence reflects honest uncertainty on borderline cases.

---

## Dataset

UCI Machine Learning Repository — E. Coli Promoter Gene Sequences
- 106 original samples (artificially reduced to 76 for imbalance demo)
- 57-character DNA sequences
- Binary classification: Promoter (+) vs Non-Promoter (-)
- Source: https://archive.ics.uci.edu/dataset/67/molecular+biology+promoter+gene+sequences

---

## Author

Vipin Gupta
vipingupta.dev@gmail.com
github.com/vipingupta-dev
