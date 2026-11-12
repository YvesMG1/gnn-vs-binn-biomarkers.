# Is "biologically informed" really informative?

**Graph neural networks, a biologically informed MLP and classical models compared on disease-severity prediction and biomarker identification from plasma proteomics.**

Course project for Deep Learning at ETH Zürich, autumn 2023, by Frederick Geiger, Yves Görgen and Hendrik Plett. Full write-up: [report.pdf](report.pdf).

## The question

Biologically informed networks wire a model along known protein–pathway relations (here from Reactome), so that each unit stands for a protein or a pathway. Hartman et al. (2023) proposed such a sparse MLP, the BINN, and reported better prediction than classical models plus biomarkers read off the trained network.

We asked three things:

1. Does a GNN on the pathway graph, which keeps the graph instead of flattening it into layers, predict better than the BINN?
2. Does the pathway structure matter at all? We tested this by scrambling it and retraining.
3. Do different models agree on which proteins and pathways are the biomarkers?

## What we found

- **No model family wins consistently.** On the acute kidney injury (AKI) hold-out set a random forest trained on protein abundances alone reached AUC 1.00, against 0.90 for the BINN and 0.94–0.97 for the GNNs. On COVID-19 the GNNs had the best hold-out AUC (0.96–0.97) but the worst cross-validation AUC. Rankings flip between cross-validation and hold-out, which at these sample sizes means the differences are within noise.
- **Scrambling the pathway structure did not hurt the BINN.** With randomised protein-to-pathway assignments and reshuffled pathway connections, its hold-out AUC went from 0.90 to 0.91 on AKI and from 0.834 to 0.835 on COVID-19.
- **Models do not agree on protein biomarkers.** On AKI, two models of the same family share 12–14 of their top 30 proteins; models of different families share 0–6. ResGCN and the random forest share none.
- **Pathway "biomarkers" follow the graph, not the disease.** ResGCN and ResGAT share 24 of their top 30 pathways on AKI, but each shares only 6 with the BINN.

On these two datasets the pathway structure gave no measurable predictive benefit, and a biomarker list read off a trained model said more about the model family than about the disease.

## Results

Cross-validation is stratified 5-fold on the training split (mean ± standard deviation). The hold-out columns are a single evaluation on the test split.

**AKI** (141 training patients, 56 hold-out patients, 554 proteins)

| Model | CV AUC | CV F1 macro | CV accuracy | Test AUC | Test F1 macro | Test accuracy |
|---|---|---|---|---|---|---|
| ResGCN | 0.975 ± 0.03 | 0.964 ± 0.04 | 0.965 ± 0.04 | 0.937 | 0.850 | 0.893 |
| ResGCN, perturbed graph | 0.976 ± 0.03 | 0.964 ± 0.03 | 0.965 ± 0.03 | 0.964 | 0.952 | 0.964 |
| ResGAT | 0.977 ± 0.03 | 0.942 ± 0.02 | 0.943 ± 0.02 | 0.968 | 0.863 | 0.911 |
| BINN | 0.999 ± 0.00 | 0.974 ± 0.02 | 0.975 ± 0.02 | 0.900 | 0.844 | 0.857 |
| BINN, perturbed pathways | 1.000 ± 0.00 | 0.966 ± 0.03 | 0.967 ± 0.03 | 0.914 | 0.865 | 0.878 |
| Random forest | 0.955 ± 0.03 | 0.748 ± 0.09 | 0.772 ± 0.07 | 1.000 | 0.976 | 0.982 |
| AdaBoost | 0.909 ± 0.10 | 0.746 ± 0.10 | 0.759 ± 0.10 | 0.947 | 0.837 | 0.875 |
| Logistic regression | 0.811 ± 0.13 | 0.689 ± 0.14 | 0.703 ± 0.14 | 0.985 | 0.905 | 0.929 |

The AKI hold-out set has 42 patients of one subphenotype and 14 of the other, so always predicting the majority class scores 0.75 accuracy, and one patient is worth 1.8 accuracy points.

**COVID-19** (707 training patients, 79 hold-out patients, 169 proteins)

| Model | CV AUC | CV F1 macro | CV accuracy | Test AUC | Test F1 macro | Test accuracy |
|---|---|---|---|---|---|---|
| ResGCN | 0.892 ± 0.04 | 0.819 ± 0.03 | 0.820 ± 0.03 | 0.959 | 0.861 | 0.861 |
| ResGCN, perturbed graph | 0.922 ± 0.02 | 0.854 ± 0.03 | 0.854 ± 0.03 | 0.896 | 0.822 | 0.823 |
| ResGAT | 0.866 ± 0.04 | 0.785 ± 0.04 | 0.789 ± 0.03 | 0.971 | 0.848 | 0.848 |
| BINN | 0.948 ± 0.01 | 0.880 ± 0.03 | 0.882 ± 0.03 | 0.834 | 0.835 | 0.835 |
| BINN, perturbed pathways | 0.952 ± 0.01 | 0.871 ± 0.02 | 0.872 ± 0.02 | 0.835 | 0.835 | 0.835 |
| Random forest | 0.949 ± 0.02 | 0.867 ± 0.02 | 0.869 ± 0.02 | 0.917 | 0.823 | 0.823 |
| AdaBoost | 0.927 ± 0.02 | 0.851 ± 0.03 | 0.854 ± 0.03 | 0.895 | 0.797 | 0.797 |
| Logistic regression | 0.909 ± 0.04 | 0.817 ± 0.04 | 0.822 ± 0.03 | 0.893 | 0.821 | 0.823 |

**Shared top-30 proteins between models, AKI**

| | ResGCN | ResGCN pert. | ResGAT | BINN | Random forest | AdaBoost |
|---|---|---|---|---|---|---|
| ResGCN | 30 | 12 | 14 | 3 | 0 | 0 |
| ResGCN pert. | 12 | 30 | 8 | 5 | 1 | 1 |
| ResGAT | 14 | 8 | 30 | 5 | 6 | 4 |
| BINN | 3 | 5 | 5 | 30 | 5 | 5 |
| Random forest | 0 | 1 | 6 | 5 | 30 | 14 |
| AdaBoost | 0 | 1 | 4 | 5 | 14 | 30 |

Overlap is higher on COVID-19 (4–19 of 30), which has fewer and less correlated proteins. The pathway overlap tables and the correlation analysis are in the report.

## Method

- **Graph.** 2,585 Reactome pathways as nodes and 2,603 directed parent–child edges. Each patient is one graph; a node's feature vector holds the abundances of the proteins assigned to that pathway and zeros elsewhere.
- **GNNs.** A GCN or GAT layer, global max pooling and an MLP classifier, with batch normalisation, dropout and weight decay (`cust_functions/graph_networks.py`). Trained with RMSprop and early stopping on validation loss. The hold-out prediction is the ensemble of the five cross-validation models.
- **BINN.** The implementation and hyperparameters of Hartman et al. (2023), four layers.
- **Classical models.** Random forest, AdaBoost and logistic regression on the protein abundance matrix, tuned by grid search.
- **Perturbation.** Protein-to-pathway assignments are shuffled; 90% of the graph edges are removed and random ones added, leaving about 10% edge overlap with the original graph. The BINN gets the equivalent treatment on its layer structure.
- **Biomarkers.** GNNExplainer importance per patient, averaged over patients and over the five fold models, for the GNNs; the BINN's own explainer; impurity-based feature importance for the tree models. Models are compared by the overlap of their top 30 proteins or pathways.

## Limitations

- **The GNNs are shallow.** The trained configurations use a single graph layer, so each pathway only sees its direct neighbours.
- **Small, single hold-out sets.** With 56 and 79 patients and no confidence intervals, none of the hold-out differences is established.
- **Cross-validation scores are optimistic.** GNN and BINN fold scores are taken at the epoch with the best validation loss on that same fold.
- **No ground truth for biomarkers.** Agreement between models is the only criterion used; it measures stability, not correctness.

## Repository

| Path | Content |
|---|---|
| `GNN_implementation.ipynb` | Graph creation, GNN training and evaluation on AKI, COVID-19 and the perturbed graphs |
| `BINN_final.ipynb`, `BINN_randomization.ipynb`, `BINN_Explain.ipynb` | BINN training, perturbation and explanations |
| `ML_models.ipynb` | Classical baselines and their top features |
| `Explain.ipynb`, `Explain_statistics.ipynb` | GNNExplainer runs and the overlap tables |
| `corr_analysis.ipynb` | Protein correlation histograms |
| `cust_functions/` | Graph creation, model definitions, training loop, explanation helpers |
| `trained_models/`, `explanations/`, `BINN_Explanations/` | Saved fold checkpoints and importance scores |
| `aki_data/`, `covid_data/`, `binn_pertubation/` | Proteomics data, Reactome pathway files and the perturbed pathway files |

## Running it

Dependencies are declared in `pyproject.toml`, pinned to the versions the experiments ran with. Python 3.8 to 3.11:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e .
```

The per-patient graphs are cached as pickles in `graph_data/`, which is not tracked. On the first run of `GNN_implementation.ipynb`, set `load_data=False, save_data=True` in the `pytorch_graphdata` calls to build them. Building them takes a while.

## Data

- **AKI:** septic acute kidney injury plasma proteomics from the FINNAKI study (Nisula et al., 2013), 197 patients in two subphenotypes, taken with the Reactome pathway files from the public repository of Hartman et al. (2023).
- **COVID-19:** plasma proteomics from Demichev et al. (2021), 786 patients grouped by WHO severity, shared with us by the authors of Hartman et al. (2023).

## References

- Hartman, E. et al. Interpreting biologically informed neural networks for enhanced proteomic biomarker discovery and pathway analysis. *Nature Communications* 14, 5359 (2023).
- Elmarakeby, H. A. et al. Biologically informed deep neural network for prostate cancer discovery. *Nature* 598, 348–352 (2021).
- Demichev, V. et al. A time-resolved proteomic and prognostic map of COVID-19. *Cell Systems* 12, 780–794 (2021).
- Nisula, S. et al. Incidence, risk factors and 90-day mortality of patients with acute kidney injury in Finnish intensive care units: the FINNAKI study. *Intensive Care Medicine* 39, 420–428 (2013).
- Ying, R. et al. GNNExplainer: generating explanations for graph neural networks. *NeurIPS* (2019).

BINN code is adapted from the [BINN documentation](https://infectionmedicineproteomics.github.io/BINN/); `util_for_examples.py` is taken from it unchanged.
