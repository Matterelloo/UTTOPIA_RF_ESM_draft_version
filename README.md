
-----

# Universal Translators of Transcriptomes to Protein Interactomes Algorithms (UTTOPIA)

> **Tools for estimating protein-protein interaction binding affinity and bound fraction from RNA-seq–derived transcript levels**

-----

## Overview

Protein–protein interaction (PPI) networks provide a powerful representation of cellular states, as variations in binding affinities and complex abundances reflect differences in cell type, structure, and pathway regulation. However, systematically measuring the full spectrum of PPIs — especially transient and weak interactions — across diverse cell types and tissues remains impractical compared to quantifying mRNA levels via RNA-seq.

**UTTOPIA** aims to partially address this gap by using minimal machine learning models to estimate PPI fraction bound ([AB]/[A]) and binding affinity (Kd) from context- and environment-specific transcriptomic data in human and yeast, achieving near-experimental accuracy.

<img src="1.png" width="400" alt="UTTOPIA Diagram">

-----

## Input Features

To achieve accurate estimations, UTTOPIA integrates multiple biological data layers:

  * **mRNA levels** of gene A and B
  * **Protein localization** (cellular compartments)
  * **ESM2 vector embedding** of genes A and B
  * **mRNA co-expression** of A and B
  * **STRING score**

**Accompanying Manuscript:**

> *(insert paper link)*

-----

## Installation

To run the UTTOPIA pipeline on your local machine, clone the repository and install the required Python packages. We recommend using a virtual environment.

```bash
git clone https://github.com/Matterelloo/UTTOPIA_RF_ESM.git
cd UTTOPIA_RF_ESM
pip install -r requirements.txt
```

> **Note:** On some systems (particularly macOS or Linux where both Python 2 and Python 3 are installed) you may need to use `pip3` instead of `pip`. On Windows, `pip` works directly from the Anaconda Prompt or any terminal where Python 3 is the default.

### Requirements

The following Python packages are required and will be installed automatically via `requirements.txt`:

```
pandas
numpy
scipy
scikit-learn
joblib
huggingface_hub
```

-----

## Datasets

All reference datasets (co-expression, STRING interaction scores, subcellular localisation, ESM2 embeddings, and pre-trained model files) for both Human and Yeast are hosted on **Hugging Face**:

> 🤗 [https://huggingface.co/datasets/caioo61/UTTOPIA-RF-ESM](https://huggingface.co/datasets/caioo61/UTTOPIA-RF-ESM)

**You do not need to download them manually.** All required files are automatically downloaded from Hugging Face and saved to your working directory when running the notebooks. This requires an internet connection on the first run; subsequent runs will use the locally cached files.

-----

## Usage

The pipeline consists of two steps, each handled by a dedicated notebook.

### Step 1 — Dataset Creation

Prepare the input feature matrix for your organism of interest by running the appropriate notebook:

  * `Dataset_Creation_Human.ipynb` — for Human data
  * `Dataset_Creation_Yeast.ipynb` — for Yeast data

These notebooks integrate your mRNA abundance data with co-expression, STRING interaction scores, and subcellular localisation, and automatically download all required reference files from Hugging Face. The output is saved as `input_to_RF_Human.csv` or `input_to_RF_Yeast.csv` in your working directory.

UTTOPIA supports three primary input formats depending on your data:

#### Table 1: User-Defined Co-expression

Ideal for users who have their own pre-calculated co-expression metrics.

  * **Input:** `gene_A, gene_B, mRNA_A, mRNA_B, coex`

#### Table 2: Default Co-expression

Ideal for targeted pairs where you want the tool to use default co-expression databases.

  * **Input:** `gene_A, gene_B, mRNA_A, mRNA_B` *(co-expression is automatically inferred)*

#### Table 3: All Possible Pairs

Ideal for broad network estimations. Input a list of genes and transcript abundances; UTTOPIA calculates metrics for all pairwise interactions.

  * **Input:** `gene, mRNA`

### Step 2 — Prediction

Run `UTTOPIA.ipynb` to generate predictions. At the top of the notebook, set the two configuration parameters:

```python
ORGANISM = "Human"   # Choose: "Human" or "Yeast"
TARGET   = "Kd"      # Choose: "Kd" or "int_stoich"
```

The notebook will automatically download the correct pre-trained Random Forest model and PCA transformer from Hugging Face, and produce a file named `UTTOPIA_Predictions_{ORGANISM}_{TARGET}.csv` in your working directory.

-----

## Development

UTTOPIA was developed as a three-stage computational pipeline to predict quantitative PPI properties — dissociation constant (Kd) and interaction stoichiometry (int\_stoich) — directly from protein sequences and transcriptomic features.

**Stage 1 — Protein Sequence Retrieval**
All protein sequences were retrieved from the reviewed UniProt proteome (Human: taxonomy ID `9606`; Yeast: taxonomy ID `4932`) via the UniProt REST API. Gene symbols were mapped to canonical amino acid sequences; known obsolete identifiers were corrected prior to mapping. Interactions where at least one sequence could not be resolved were excluded.

**Stage 2 — ESM-2 Sequence Embeddings**
Each protein sequence was embedded using **ESM-2** (`esm2_t6_8M_UR50D`), a protein language model trained on \~250 million protein sequences from UniRef50. Embeddings were computed via mean pooling over the final hidden layer, producing a 320-dimensional vector per protein encoding structural and functional information.

**Stage 3 — PCA Dimensionality Reduction & Random Forest**
For each protein pair (Cmin, Cmax), ESM-2 embeddings were independently compressed using Principal Component Analysis (20 components per protein, 40 per pair). The resulting PCA vectors were concatenated with co-expression and subcellular localisation features to form the final feature set. A **Random Forest regressor** (hyperparameters selected via GridSearchCV) was trained separately on human and yeast datasets to predict Kd or stoichiometry. PCA was fitted on the full protein embedding matrix before train/test splitting; since embeddings carry no label information, this does not constitute data leakage. The Random Forest was trained strictly on the training split.

-----

## References

### Training Data

1.  **Cho, N. H. et al. (2022)**. OpenCell: Endogenous tagging for the cartography of human cellular organization. *Science*, 375, eabi6983.
2.  **Hein, M. Y. et al. (2015)**. A Human Interactome in Three Quantitative Dimensions Organized by Stoichiometries and Abundances. *Cell*, 163, 712–723.
3.  **Michaelis, A. C. et al. (2023)**. The social and structural architecture of the yeast protein interactome. *Nature*, 624, 192–200.

### Methods & Tools

  * **UniProt:** UniProt Consortium. *UniProt: the Universal Protein Knowledgebase in 2023.* Nucleic Acids Research, 51(D1), D523–D531 (2023). [https://doi.org/10.1093/nar/gkac1052](https://doi.org/10.1093/nar/gkac1052)
  * **ESM-2:** Lin, Z. et al. *Evolutionary-scale prediction of atomic-level protein structure with a language model.* Science, 379(6637), 1123–1130 (2023). [https://doi.org/10.1126/science.ade2574](https://doi.org/10.1126/science.ade2574)
  * **scikit-learn:** Pedregosa, F. et al. *Scikit-learn: Machine Learning in Python.* JMLR, 12, 2825–2830 (2011). [https://jmlr.org/papers/v12/pedregosa11a.html](https://jmlr.org/papers/v12/pedregosa11a.html)
  * **Compartments:** Subcellular localization databases
  * **STRING:** Protein–Protein Interaction Networks

-----

## License

This project is released under the **MIT License**.
