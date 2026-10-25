# Explanation Reliability under Distribution Shift in Anomaly Detection

Code for the paper **"Explanation Reliability under Distribution Shift in Anomaly Detection: A Stability–Faithfulness Evaluation Framework"** (Kumar, Dwivedi, & Kilickaya; submitted to the *Journal of Artificial Intelligence Research*).

The study asks whether feature-attribution explanations of anomaly detectors stay **stable** and **faithful** when the data distribution shifts after deployment. It evaluates three detectors (autoencoder, variational autoencoder, Isolation Forest) and two attribution methods (Integrated Gradients, KernelSHAP) on credit-card fraud and CIC-IDS2017 intrusion data. A second, independent in-distribution holdout (ID2) separates shift effects from sampling variance, and all differences are tested with Mann–Whitney U tests and Benjamini–Hochberg false-discovery-rate correction.

## Main findings

1. **Stability degrades under shift, and the degradation is class-specific.** 16 of 30 comparisons are significant: CIC-IDS2017 attacks in 10 of 10 model–method–shift combinations, benign credit-card transactions in 4 of 5, credit-card frauds in 0 of 5.
2. **For CIC-IDS2017 attacks, the effect holds for every detector, attribution method, and shift scenario.**
3. **A no-shift baseline is necessary.** Independent in-distribution samples agree only modestly (mean Spearman 0.08–0.50).
4. **Unnormalised deletion-based faithfulness follows anomaly-score magnitude under shift.** It reports an apparent gain where attribution rankings have become unstable, so it cannot certify explanation reliability.

## Repository structure

The pipeline was run as Kaggle notebooks (NVIDIA Tesla T4). Each notebook reads the outputs of earlier ones from Kaggle datasets; run them in this order.

| Notebook | Purpose | Main outputs |
|---|---|---|
| `NB0.ipynb` | Data cleaning, distribution-shift construction, ID/ID2/OOD partitioning, standardisation | Split parquet files, scalers, `config.json` per dataset |
| `NB0D1A.ipynb` | Autoencoder and initial VAE training, credit card | Models and anomaly scores |
| `NB1cont.ipynb` | Autoencoder training, CIC-IDS2017 scenario_T | Model and anomaly scores |
| `NB1.ipynb` | VAE training (all settings, global output variance) and autoencoder, scenario_A | Models and anomaly scores |
| `NB2.ipynb` | Deep SVDD (exploratory; **not used in the paper**, dropped after persistent hypersphere collapse) | — |
| `NB3.ipynb` | Isolation Forest training (all settings) | Models and anomaly scores |
| `NB4.ipynb` | Integrated Gradients attributions for the stability analysis | Attribution arrays |
| `NB5.ipynb` | KernelSHAP attributions for the stability analysis | SHAP value arrays |
| `NB6.ipynb` | Stability (paired Spearman) and faithfulness (deletion-AUC) computation | Per-pair and per-instance metric arrays, result tables |
| `nb7_8.ipynb` | Significance testing (Mann–Whitney U, BH-FDR, rank-biserial effect sizes), paper tables and figures | Tables 2–4, figures |

## Data

The datasets are public and are **not** redistributed here.

- **Credit card fraud** (Dal Pozzolo et al., 2015): https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud
- **CIC-IDS2017** (Sharafaldin et al., 2018): https://www.unb.ca/cic/datasets/ids-2017.html. We used the eight daily flow files of the machine-learning release, as mirrored at https://www.kaggle.com/datasets/chethuhn/network-intrusion-dataset.

On Kaggle, add both datasets to `NB0.ipynb` with **+ Add Data**. `NB0` locates the files automatically.

### Distribution-shift settings

| Setting | In-distribution pool | OOD set |
|---|---|---|
| `credit_card` | Day 1 (Time < 86,400 s) | Day 2 |
| `scenario_T` (temporal) | CIC-IDS2017 Monday–Wednesday | Thursday–Friday |
| `scenario_A` (unseen attack types) | CIC-IDS2017 Monday–Thursday | Friday (DDoS, PortScan, Bot never seen) |

In-distribution benign data are split 70/15/15 into training, ID, and ID2 subsets; in-distribution anomalies are split 50/50 between ID and ID2, so no anomaly is used for training.

## Key settings

| Component | Setting |
|---|---|
| Autoencoder / VAE | Adam, lr 1e-3, batch 256, early stopping (patience 10, min delta 1e-5) on a 10% benign validation split |
| VAE | Global scalar output variance; latent log-variance clamped to [−10, 10]; score = Gaussian reconstruction NLL |
| Isolation Forest | 100 trees, `max_samples="auto"`, `contamination="auto"` |
| Integrated Gradients | Zero baseline (training mean), 300 steps, convergence-error filter |
| KernelSHAP | 50 benign background instances from the ID split, `nsamples="auto"` |
| Deletion-AUC | Top-\|attribution\| features masked to 0 at fractions 0, 0.1, …, 1; trapezoidal AUC |
| Random seed | 42 throughout |

## Installation

```bash
git clone https://github.com/OzlemGulsumKilickaya/stable-xai-anomaly.git
cd stable-xai-anomaly
pip install -r requirements.txt
```

The notebooks were developed on Kaggle, where most dependencies are preinstalled; `captum` and `shap` are installed inside the notebooks if missing.

## Citation

If you use this code, please cite:

```bibtex
@article{kumar2026explanation,
  title   = {Explanation Reliability under Distribution Shift in Anomaly Detection:
             A Stability--Faithfulness Evaluation Framework},
  author  = {Kumar, Basant and Dwivedi, Rashmi and Kilickaya, Ozlem},
  journal = {Journal of Artificial Intelligence Research},
  year    = {2026},
  note    = {Under review}
}
```

## License

Released under the MIT License (see `LICENSE`). The datasets remain subject to their original licences.

## Contact

Ozlem Kilickaya — ozlem.kilickaya@uopeople.edu
