# Tail-Sensitive Leakage Assessment for ML-KEM

**Combining Extreme Value Theory with Deep Learning-Based Profiled Attacks**

M.Tech (Cyber Security) dissertation, MACSE698 Dissertation-1, VIT Vellore, 2026.
Author: G Madhusudan · Guide: Dr. Jeyanthi N, SCORE, VIT Vellore.

> This repository contains my dissertation work done at VIT Vellore under the guidance of Dr. Jeyanthi N.
> It is not an official VIT publication.

---

## What this project does

ML-KEM (FIPS 203) is the NIST post-quantum key-encapsulation standard. This project measures how three
ML-KEM implementations leak the **Hamming weight (HW) of secret coefficient 0** through electromagnetic (EM)
emissions during the NTT-domain base multiplication. It uses the public Ledger Donjon EM dataset and does the following:

1. **Leakage assessment:** Welch t-test and SNR on the complete profiling sets.
2. **EVT cut-off for points of interest:** a Generalised Pareto Distribution (GPD) fitted to the top 2% of |t|, compared with a fixed top-50 rule.
3. **Deep-learning profiled attacks:** MLP and CNN models, evaluated on validation data, an untouched test split and the **independent fixed-key attack sets**.
4. **Diagnosis** of an attack that works on validation data but fails on the attack recordings.

The pipeline has 11 stages. The full method, results and limitations are in [`docs/final_report.pdf`](docs/final_report.pdf).

## Main results

| | pqm4 (unprotected) | ref (unprotected) | mkm4 (first-order masked) |
|---|---|---|---|
| Max \|t\| on the full profiling set (sample) | 168.39 (183) | 55.19 (7917) | 3.20 (1647): below 4.5 |
| Points of interest: top-50 → GPD cut-off | 50 → 27 | 50 → 15 | 50 → 50 |
| Stage 11 validation accuracy, K = 50 (majority baseline) | 39.61% MLP (23.61%) | 94.13% CNN (23.37%) | not attacked |
| Accuracy on the full attack set | 32.38%: fails, decides HW 5 (true HW 6) | **94.65%**, success rate 0.892 at 1 trace and 1.0 at 10 traces | — |

- **Masked implementation:** no first-order leakage of the secret. As a positive control, one share alone gives |t| = 262.83.
- **pqm4 failure:** traced to a shift between the profiling and attack recording sessions. Attack features are offset by 0.12 standard deviations, about 21× the validation level, and the model's expected HW drops from about 5.9 to 5.27. It is not caused by the model or by the secret value.
- **GPD cut-off:** a data-driven stopping rule, not a calibrated statistical test. The score keeps the |t| ranking, so only the threshold u decides the cut, and the fitted shape ξ depends on the tail fraction (see the report, Section 6.1).

## Repository layout

```
notebooks/
  01_capstone_stages1-11.ipynb   full pipeline, Stages 1–11, with outputs (Google Colab)
  02_final_review_demo.ipynb     short demo used at the final review (with outputs)
results/                         saved .npz arrays (t-test/SNR, POIs, EVT scores, GE curves, Stage 11 files)
models/                          trained Stage 8 and Stage 11 Keras models
figures/                         plots and diagrams used in the report and slides
docs/
  final_report.pdf               dissertation report
  final_review_slides.pptx/.pdf  final review presentation
```

## Dataset

The raw traces are **not included** (about 13 GB). Download them from the Ledger Donjon open ML-KEM EM dataset:

> A. A. Magazin and K. M. Abdellatif, "Open EM Side-Channel Dataset for ML-KEM (Kyber) Implementations,"
> IACR Cryptology ePrint Archive, 2026/1851.

The notebook expects the extracted dataset at `/content/drive/MyDrive/mlkem_dataset/extracted/d0nj0n_mlkem_dataset/`.
Some intermediate feature files (e.g. `cnn_features_full.npz`, about 130 MB) exceed GitHub's size limit and are not included; the notebook regenerates them.

## Running

The notebooks were written for **Google Colab** with Google Drive mounted; Stage 11 used an NVIDIA L4 GPU.

```bash
pip install -r requirements.txt
```

Then open the notebooks in Colab (or Jupyter), set the dataset path, and run them in order. The 8 GB pqm4 trace files are processed in 10,000-trace chunks.

## Scope and limitations

- One coefficient (0) and one leakage model (Hamming weight): the attack recovers a HW class, not the full key.
- The pqm4 session shift is diagnosed, not corrected.
- The masked implementation is analysed at first order only.
- The research questions, limitations and future work are discussed in Chapters 7–8 of the report.

## Acknowledgements

Dissertation guide: Dr. Jeyanthi N, School of Computer Science Engineering and Information Systems, VIT Vellore.
Dataset: Ledger Donjon. The report's diagrams were drawn with AI assistance (Anthropic, 2026), as noted in Section 4.1.6 of the report.

## Licence

Code is released under the MIT Licence (see `LICENSE`). The dataset is not part of this repository and remains under its original licence.
