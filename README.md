# Neural Network Inverse Design of Mechanical MNIST Geometries

**Author:** Andy Kapoor

## Repository Structure
```
.
├── Mech_MNIST_InverseDesign.ipynb    # Main analysis notebook
├── data/
│   ├── EMNIST-1.zip                  # EMNIST letters for generalization testing
│   ├── mnist_img_train.txt.zip       # MNIST training bitmap data
│   └── mnist_img_test.txt.zip        # MNIST test bitmap data
└── README.md
```

> **Note:** The `FEA_displacement_results_step12/` folder is not included in this repository due to size. Download it from the Mechanical MNIST GitHub: https://github.com/elejeune11/Mechanical-MNIST and place it in the root directory before running the notebook.

---

## Overview

This project tackles an inverse design problem using the Mechanical MNIST dataset: given a finite element analysis (FEA) displacement field as input, reconstruct the original 28x28 bitmap image that produced it. A multilayer perceptron (MLP) is trained on MNIST handwritten digits to learn this mapping from mechanical response back to geometry. The trained model is then tested on EMNIST letter data (geometries it was never trained on) to evaluate out-of-distribution generalization without transfer learning or fine-tuning.

---

## Dataset

**Mechanical MNIST** is a benchmark dataset introduced by Lejeune (2020) consisting of 70,000 finite element simulations of heterogeneous material domains under large deformation. Each sample pairs an MNIST bitmap image (used to define material properties on a 28x28 grid) with the resulting FEA displacement fields. This project uses the displacement outputs at step 12 as model inputs and recovers the original bitmap as the target output.

- Paper: Lejeune, E. (2020). Mechanical MNIST: A benchmark dataset for mechanical metamodels. *Extreme Mechanics Letters*. https://doi.org/10.1016/j.eml.2020.100659
- Dataset: https://open.bu.edu/handle/2144/39371
- GitHub: https://github.com/elejeune11/Mechanical-MNIST

---

## Problem

In forward FEA, a geometry (bitmap) produces a displacement field. The inverse problem asks: given only the displacement field, can we recover the original geometry? This has practical relevance in structural and materials design, where observed mechanical responses need to be mapped back to the underlying structure that caused them.

---

## Data

| Dataset | Description |
|---|---|
| MNIST digits | 60,000 train / 10,000 test 28x28 handwritten digit bitmaps |
| FEA displacement fields | x and y displacement outputs at 784 nodes per image (1,568 features total), computed at step 12 |
| EMNIST letters | 10 handwritten letters (G, J, K, M, N, O, P, Q, S, X) used for out-of-distribution testing |

---

## Methods

### Model Architecture
- **MLP Regressor** (scikit-learn): 3 hidden layers (256 → 128 → 64 neurons)
- Activation: ReLU
- Optimizer: Adam
- Learning rate: 0.005
- Early stopping: patience of 20 epochs, tolerance 1e-5
- Input: 1,568-feature displacement field (x and y concatenated), StandardScaler normalized
- Output: 784-feature flattened bitmap (28x28 pixel intensities)

### Training
- Fit on 60,000 MNIST training samples
- Converged after 39 epochs
- MSE tracked per epoch to monitor training loss in bitmap intensity units squared

### Evaluation
- MNIST test set: standard held-out evaluation
- EMNIST letters: zero-shot generalization test on unseen character geometries using the same trained model and scaler

---

## Results Summary

| Dataset | MAE (bitmap intensity units) |
|---|---|
| MNIST test set | low (model trained on this distribution) |
| EMNIST letters | ~3x higher than MNIST |

The model successfully recovers the global shape and form of MNIST digits from displacement fields alone, though fine edge detail is lost. On EMNIST letters, coarse structural features are partially recoverable (e.g. highlights in X, G, K), but significant noise appears and detail is lost. This is expected given the model was never exposed to letter geometries.

---

## Dependencies
```
numpy
matplotlib
scipy
scikit-learn
torch
tqdm
```

Install with:
```bash
pip install numpy matplotlib scipy scikit-learn torch tqdm
```

---

## Limitations & Future Work

- Fine detail in reconstructed bitmaps is lost. Higher-capacity models or convolutional architectures may improve resolution.
- EMNIST generalization is poor without retraining. Adding letter samples or fine-tuning on mixed data would improve transferability.
- Only a single FEA step (step 12) was used. Incorporating multi-step displacement histories could provide richer input features.
- Future work: convolutional inverse design networks, physics-informed constraints, transfer learning across character classes.
