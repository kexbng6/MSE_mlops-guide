# MLOps - Celestial Body Classification

This repository contains the code from [A guide to MLOps](https://mlops.swiss-ai-center.ch/).

## Progress
- ✅ **Chapter 1.1**: Run a simple ML experiment with Jupyter Notebook
- ✅ **Chapter 1.2**: Adapt and move the Jupyter Notebook to Python scripts
- ✅ **Chapter 1.3**: Initialize Git and DVC for local training
- ✅ **Chapter 1.4**: Reproduce the ML experiment with DVC

## Chapter 1.2 - Adapt and Move the Jupyter Notebook to Python Scripts

This chapter focuses on transforming a Jupyter Notebook into modular Python scripts for better reproducibility, automation, and maintainability.

## Results

### Dataset Preparation
- **Total images**: 1500 (10 classes: Earth, Jupiter, Mars, Mercury, Moon, Neptune, Pluto, Saturn, Uranus, Venus)
- **Training set**: 1200 images
- **Validation set**: 300 images
- **Output**: Saved in `data/prepared/` (includes `train/`, `val/`, `labels.json`, `preview.png`)

### Model Training
- **Architecture**: Simple CNN
  - Conv2D (32 filters, 3x3 kernel, ReLU activation)
  - MaxPooling2D (3x3)
  - Flatten
  - Dense (64 units, ReLU activation)
  - Dense (10 units, linear activation)
- **Total parameters**: 205,834
- **Training results** (5 epochs):
  | Epoch | Training Loss | Training Accuracy | Validation Loss | Validation Accuracy |
  |-------|---------------|-------------------|-----------------|----------------------|
  | 1     | 2.2275        | 15.25%            | 2.1528          | 22.67%               |
  | 2     | 2.1461        | 19.50%            | 2.0679          | 22.67%               |
  | 3     | 2.0632        | 27.17%            | 1.9759          | 22.67%               |
  | 4     | 1.9752        | 29.75%            | 1.8697          | 55.00%               |
  | 5     | 1.8811        | 40.00%            | 1.7723          | **44.33%**           |
- **Output**: Saved in `model/` (`model.keras`, `history.npy`)

### Model Evaluation
- **Validation loss**: 1.77
- **Validation accuracy**: 44.33%
- **Precision**: 0.31
- **Recall**: 0.46
- **F1 score**: 0.35
- **Output**: Saved in `evaluation/` (`metrics.json`, `plots/`)

## Reproduce the Experiment

To reproduce the results, run the following commands in order:

```bash
# 1. Prepare the dataset
python3.13 src/prepare.py data/raw data/prepared

# 2. Train the model
python3.13 src/train.py data/prepared model

# 3. Evaluate the model
python3.13 src/evaluate.py model data/prepared
```

## Notes
- **Python Version**: Use `python3.13` (as the virtual environment is configured with Python 3.13).
- **Warning**: If you use `python3.12`, you may encounter a `ModuleNotFoundError: No module named 'keras'` error. Always activate the virtual environment first:
  ```bash
  source .venv/bin/activate
  ```
- **Jupyter Lab**: To run the original Jupyter Notebook:
  ```bash
  jupyter-lab notebook.ipynb
  ```
  Access it at:
  - http://localhost:8888/lab
  - Token: `a323e2042b8f09959715aa055f7f537f5f913c100483ca1c`

## Chapter 1.3 - Initialize Git and DVC for Local Training

### Commands Executed
```bash
# Freeze dependencies
pip freeze --local --all > requirements-freeze.txt

# Initialize DVC
dvc init

# Disable DVC analytics
dvc config core.analytics false

# Track raw data with DVC (after fixing .gitignore)
dvc add data/raw

# Stage changes for Git
git add .

# Commit changes
git commit -m "Use DVC to version the data in my ML experiment"
```

### Key Changes
- **DVC Initialization**: Created `.dvc/`, `.dvcignore`, and `data/raw.dvc`.
- **Git Ignore Update**: Modified `.gitignore` to exclude `data/prepared/`, `evaluation/`, `model/`, and `.venv/`.
- **DVC Tracking**: Added `data/raw` to DVC for versioning large files.

### Notes
- **Error Fixed**: Initially, `dvc add data/raw` failed because `data/` was git-ignored. Updated `.gitignore` to only ignore `data/prepared/`.
- **Git Status**: Used `git status` (not `git stauts`) to check staged changes.

## Chapter 1.4 - Reproduce the ML Experiment with DVC

### DVC Pipeline
The DVC pipeline (`dvc.yaml`) defines the following stages:
```
+--------------+
| data/raw.dvc |
+--------------+
        *
        *
        *
  +---------+
  | prepare |
  +---------+
        *
        *
        *
    +-------+
    | train |
    +-------+
        *
        *
        *
  +----------+
  | evaluate |
  +----------+
```

### Commands Executed
```bash
# Visualize the DVC pipeline
dvc dag

# Reproduce the entire pipeline
dvc repro

# Stage changes for Git
git add .

# Commit changes
git commit -m "Use DVC to save the commands of my ML experiment"
```

### Key Changes
- **DVC Pipeline**: Created `dvc.yaml` to define the pipeline stages (`prepare`, `train`, `evaluate`).
- **DVC Lock File**: Generated `dvc.lock` to track dependencies and outputs of each stage.
- **Git Ignore Updates**: Added `.gitignore` files for `evaluation/` and `evaluation/plots/`.

### Notes
- **Pipeline Execution**: `dvc repro` automatically runs only the necessary stages (skips `data/raw.dvc` if unchanged).
- **Outputs**: All stages executed successfully, generating `data/prepared/`, `model/`, and `evaluation/`.
- **Git Status**: Used `git status` (not `git stauts`) to check staged changes.