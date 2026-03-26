# GSoC 2026 — ML4SCI GENIE Submission
**Deep Graph Anomaly Detection with Contrastive Learning for New Physics Searches**

Felipe Tomé Pereira — Pontifícia Universidade Católica do Paraná (PUCPR)

## Overview

This repository contains the implementation for the ML4SCI GENIE project submission,
including all common tasks and the specific task for the
*Deep Graph Anomaly Detection with Contrastive Learning* project.

The work demonstrates a complete pipeline for representing particle jet events as graphs
and applying graph neural networks for classification and representation learning —
the technical foundation for the proposed GSoC anomaly detection framework.


## Repository Structure

```
gsoc-ml4sci-genie-submission/
├── common_task_1.ipynb       # Autoencoder for jet image reconstruction
├── common_task_2.ipynb       # Graph-based GNN classifier (quark/gluon)
├── specific_task_1.ipynb     # SimCLR contrastive learning on jet graphs
├── models/
│   ├── best_autoencoder.pt
│   ├── best_gnn.pt
│   ├── best_CLR.pt
│   └── best_LinClassifier.pt
├── datasets/                 # cache files generated after first run
└── README.md
```


## Dataset

The dataset is not included in this repository due to size.

**Download:** [quark-gluon_data-set_n139306.hdf5](https://drive.google.com/file/d/1WO2K-SfU2dntGU4Bb3IYBp9Rh7rtTYEr/view?usp=sharing)

After downloading, place the file in the root directory or update `FILE_PATH` in each notebook.

**Dataset:** 139,306 quark/gluon jet events, each represented as a 125×125×3 image
with channels corresponding to ECAL, HCAL, and Tracker detector subsystems.


## Tasks

### Common Task 1 — Convolutional Autoencoder (`common_task_1.ipynb`)

Trains a convolutional autoencoder to learn a compressed representation of jet images.

**Pipeline:**
- Log1p transform + per-channel normalization
- Encoder: 4 conv blocks (3→16→32→64→32), bottleneck at 8×8×32
- Decoder: 4 transposed conv blocks with Dropout2d and Sigmoid output
- MSE reconstruction loss

**Results:**
- Gluon mean reconstruction error: 0.000772
- Quark mean reconstruction error: 0.000602
- Gluons have higher error — consistent with their broader, more complex jet structure


### Common Task 2 — Graph-based GNN Classifier (`common_task_2.ipynb`)

Converts jet images to graphs and trains an EdgeConv GNN for quark/gluon classification.

**Graph construction:**
- Nodes: active pixels (non-zero across any channel)
- Node features: normalized channel intensities (3) + x_norm, y_norm, r_norm (3) = 6 features
- Edges: k-NN (k=8) in normalized (η, φ) pixel space, following ParticleNet
- Edge features: dx, dy, Euclidean distance, intensity difference

**Models compared:**

| Model | AUC | Accuracy | F1 |
|-------|-----|----------|----|
| Logistic Regression (mean pool) | 0.515 | 0.512 | 0.509 |
| SmallCNN | 0.747 | 0.687 | 0.697 |
| EdgeConv GNN | **0.779** | **0.712** | **0.717** |


### Specific Task 1 — SimCLR Contrastive Learning (`specific_task_1.ipynb`)

Implements a two-phase contrastive learning pipeline for quark/gluon classification.

**Phase 1 — SimCLR pretraining (unsupervised):**
- Graph augmentations: node feature dropout (p=0.1) + edge dropout (p=0.1)
- GNNEncoder: 2 EdgeConv layers + dual pooling (mean + max)
- ProjectionHead: 2-layer MLP with BatchNorm
- NT-Xent loss with temperature τ=0.5
- Trained on stratified subset of 20,000 events

**Phase 2 — Linear probe (supervised):**
- Encoder frozen after pretraining
- Single linear layer trained on top for classification

**Results:**

| Model | AUC | Accuracy | F1 |
|-------|-----|----------|----|
| SimCLR + linear probe | 0.632 | 0.597 | 0.561 |
| EdgeConv supervised (Task 2) | 0.779 | 0.712 | 0.717 |

**Note on hardware:** Full SimCLR convergence requires GPU. Training was conducted on CPU,
limiting convergence (NT-Xent loss ~4.40 vs ideal ~2.0-2.5). For the GSoC project,
training will be conducted on Google Colab (T4/A100), expected to reduce training time
by 10-20x and allow proper convergence on the full dataset.


## Installation
```bash
git clone https://github.com/felipetp-ctrl/gsoc-ml4sci-genie-submission
cd gsoc-ml4sci-genie-submission
pip install -r requirements.txt
```

PyTorch Geometric installation may require additional steps depending on your CUDA version.
See the [official documentation](https://pytorch-geometric.readthedocs.io/en/latest/install/installation.html).


## Running the Notebooks

Run in order:

```
1. common_task_1.ipynb
2. common_task_2.ipynb   # generates graph cache on first run (~30 min)
3. specific_task_1.ipynb # loads graph cache from task 2
```

The graph cache (`datasets/qg_graph_dataset_k8_edgeattr.pt`) is generated automatically
on the first run of `common_task_2.ipynb` and reused by `specific_task_1.ipynb`.


## Connection to the GSoC Project

This submission demonstrates the technical foundation for the proposed
*Deep Graph Anomaly Detection with Contrastive Learning* project:

- **Graph construction pipeline** (Task 2) → reused for LHC Olympics 2020 and Top Quark Tagging datasets
- **Convolutional autoencoder** (Task 1) → reconstruction baseline for anomaly detection
- **SimCLR on graphs** (Specific Task 1) → prototype of the contrastive anomaly detection framework

In the full project, the SimCLR encoder would be trained on QCD background jets only,
and anomaly scores would be computed as the distance of test embeddings from the learned
normal distribution — following the approach of Luo et al. (2022).

## Author

**Felipe Tomé Pereira**
- GitHub: [felipetp-ctrl](https://github.com/felipetp-ctrl)
- LinkedIn: [felipetpereira](https://linkedin.com/in/felipetpereira)
- Email: felipetpereira1@gmail.com
