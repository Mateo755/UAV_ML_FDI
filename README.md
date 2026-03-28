# PADRE - UAV_measurement_data

The repository contains sensory data from drones collected during flights with different types of propeller failures. Measurements from accelerometers, gyroscopes, microphones and other sensors were collected during flights with different types of failures occurring in various configurations in one, two, three or four rotors. Raw sensor output, preprocessed data and digitally processed signals have been made available. The shared repository should be helpful in developing methods for detecting and classifying defects in unmanned aerial vehicle (UAV) actuators. It will be particularly useful for researchers working on data-driven methods. The default purpose of the dataset is to train artificial intelligence models that require large amounts of data.

The names of the main directories indicate the construction of the UAV from which the data was collected. The subsequent folders include Readme.txt files detailing the contents of each folder.

More information about the project can be found at:

AeroLAB website: https://uav.put.poznan.pl/archives/763,
https://uav.put.poznan.pl/archives/788

JINT paper: https://link.springer.com/article/10.1007/s10846-024-02101-7

ICUAS paper: https://ieeexplore.ieee.org/document/10156238

SII paper: https://ieeexplore.ieee.org/document/10417427

---

# Project Report

### 1 Objectives and Scope

The primary goal of the research was to develop and verify a propeller fault detection algorithm for the Parrot Bebop 2 unmanned aerial vehicle. The following research tasks were carried out within the project:

1. **Data pipeline:** Implementation of modules responsible for formatting prepared telemetry data (the PADRE repository) into input vectors for the neural network.
2. **Deep learning models:** Design and training of diverse deep neural network architectures tailored to the characteristics of the analyzed signals.
3. **Comparative evaluation:** Testing model performance on an independent hold-out test set to identify the architecture with the highest classification quality metrics.

### 2 Data Characteristics and Processing

#### 2.1 Data representation

The project used two complementary approaches to representing sensory data, using dedicated subsets of the PADRE repository:

- **Time domain:** Input data were taken from the `Normalized_data` directory.
- **Frequency domain:** Data were obtained from the `FFT_data` directory. In the experiments reported here, analysis was limited to data processed with a **Hann** window.

#### 2.2 Dataset split

To ensure credible results and avoid overfitting, the dataset was split as follows:

1. **Training set (70%):** used to optimize network weights.
2. **Validation set (15%):** used to monitor the learning process.
3. **Test set (15%):** a held-out set used **only** for final model evaluation.

The data were balanced so that each fault state was equally represented across the splits.

### 3 Experiment: Five-Class Classification

The main experiment reduced the output space to **five fundamental classes**. This aggregation allows the operator to quickly assess the state of the drone without detailed analysis (which is addressed by the 20-class model).

#### 3.1 Definition of decision classes

The labeling logic is based on counting how many rotors exhibit anomalies, regardless of their physical placement or degradation level (level 1 or 2).

- **Class 0 – Nominal (NO):** Normal flight. All four rotors operate correctly.  
  _(Scenario code: `"0000"`)_

- **Class 1 – Single rotor failure:** Aggregates all cases in which exactly one propulsion unit failed. This includes damage to motor A, B, C, or D.  
  _(Example codes: `"1000"`, `"0020"`, `"0001"`)_

- **Class 2 – Double rotor failure:** Scenarios in which anomalies were detected on two rotors simultaneously.  
  _(Example codes: `"1100"`, `"1002"`, `"0120"`)_

- **Class 3 – Triple rotor failure:** Critical scenarios with three damaged rotors.  
  _(Example codes: `"1120"`, `"0122"`)_

- **Class 4 – Quadruple rotor failure:** Total degradation of the propulsion system, with anomalies on all four rotors.  
  _(Scenario code: `"1122"`)_

#### 3.2 Implemented network architectures

Two different neural network architectures were evaluated for signal classification. Both were implemented in an environment that allows straightforward parametrization (e.g., number of layers, neurons, or filters).

- **MLP (Multi-Layer Perceptron)**
- **CNN (1D Convolutional Neural Network)**

#### 3.3 Results and Evaluation

**Table 1:** Top 10 models by test-set accuracy. The highest scores were achieved by architectures operating **only in the time domain (Time)**; models using frequency features (**FFT**) did **not** rank in the top ten.

| Trial | Model type | Epoch | Test accuracy | Test F1 score |
| ----- | ---------- | ----- | ------------- | ------------- |
| v25   | CNN        | 50    | 0.98379       | 0.98037       |
| v27   | CNN        | 38    | 0.98332       | 0.98024       |
| v29   | CNN        | 30    | 0.98202       | 0.97795       |
| v30   | CNN        | 40    | 0.98202       | 0.97986       |
| v26   | CNN        | 35    | 0.98143       | 0.97606       |
| v24   | CNN        | 43    | 0.98115       | 0.97699       |
| v28   | CNN        | 25    | 0.98103       | 0.97508       |
| v17   | MLP        | 88    | 0.97746       | 0.97184       |
| v22   | MLP        | 86    | 0.97607       | 0.96924       |
| v21   | MLP        | 67    | 0.97593       | 0.97110       |

**Evaluation metrics used:**

- **Test accuracy:** The fraction of correctly classified test samples. A value of 0.98379 means the model identified the drone’s state correctly in **98.38%** of cases.
- **Test F1-score:** The harmonic mean of precision and recall. It is particularly important in safety-related FDI systems, where the cost of a missed fault (false negative) is high. A high F1 indicates that the model balances detection of faults without generating excessive false alarms.

![Training and validation accuracy for the best architecture, Time_CNN_v25.](https://raw.githubusercontent.com/Mateo755/UAV_ML_FDI/main/assets/report/fig01a_accuracy_time_cnn_v25.png)

_Figure 1 (a): Accuracy — learning curves for the best architecture (Time_CNN_v25)._

![Training and validation loss for the best architecture, Time_CNN_v25.](https://raw.githubusercontent.com/Mateo755/UAV_ML_FDI/main/assets/report/fig01b_loss_time_cnn_v25.png)

_Figure 1 (b): Loss — training and validation loss for the same model. Together, (a) and (b) illustrate a stable learning process without significant overfitting._

**Table 2:** Five-class classification results (test set).

| Metric            | Value   | Comment                  |
| ----------------- | ------- | ------------------------ |
| Accuracy          | 0.98379 | High overall performance |
| Precision (macro) | 0.9887  | Detection precision      |
| Recall (macro)    | 0.9725  | Model sensitivity        |
| F1-score          | 0.9804  | Harmonic mean            |

#### 3.4 Confusion Matrix

A confusion matrix helps interpret the types of errors the model makes.

![Confusion matrix for the best model on the test set.](https://raw.githubusercontent.com/Mateo755/UAV_ML_FDI/main/assets/report/fig02_confusion_matrix_test.png)

_Figure 2: Confusion matrix for the best model on the test set._

**Result analysis:** The confusion matrix (Fig. 2) shows high classifier performance in estimating fault severity. Dominant values on the diagonal confirm that the model correctly distinguishes the number of faulty propulsion units—from a single failure through total loss of all four rotors.

Off-diagonal errors show a specific tendency: in ambiguous cases the model tends to predict **class 1** (single failure). For example, among **9484** correctly identified cases of two-rotor damage (class 2), **169** errors underestimated the fault as class 1. This margin is nevertheless below **1.8%**, indicating strong separability between degradation levels.

### 4 Summary and Conclusions

The experiments confirmed that machine learning can effectively detect and assess fault severity in unmanned aerial vehicles. The best trained model reached **98.38%** classification accuracy on the test set, suggesting strong application potential.

Key conclusions from comparing architectures and signal domains:

- **Time domain dominance:** Contrary to initial assumptions, models using raw time-domain statistics achieved **higher** scores than those using the processed frequency spectrum (FFT). This suggests that information about fault multiplicity is contained in signal dynamics (e.g., sudden acceleration changes), not only in vibrational characteristics.
- **Advantage of CNNs:** CNN architectures (especially Time_CNN variants) showed **higher** effectiveness and training stability than classical MLPs. Convolutional networks better extracted local patterns from time series, achieving a higher F1-score in **fewer** training epochs.

A more detailed description of five-state classification and further work—including **20-class** detection and automated hyperparameter search—are covered in the appendices.

### 5 Project Development Perspectives

Although results are satisfactory (accuracy **> 98%**), the project opens paths toward industrial deployment. Potential directions include:

- **Edge AI:** Model optimization (quantization, pruning) and deployment on microcontrollers or onboard computers (e.g., NVIDIA Jetson Nano, Raspberry Pi) to validate inference latency in real control loops.
- **Time–frequency fusion:** Time and frequency domains were compared separately. A valuable next step is a **multi-input** architecture learning correlations between time dynamics and spectral (FFT) features, potentially improving harder cases (e.g., 20-class settings).
- **Validation under varying environmental conditions:** Extending the training set with flights in harsh weather (strong wind, turbulence) to check whether the model confuses natural flight disturbances with rotor faults (reducing false positives).
- **Anomaly detection (unsupervised learning):** Moving beyond supervised classification toward anomaly detection would allow identifying faults not defined in the training set.

### Appendix A: Detailed interactive W&B report

Full project documentation and experiments—including interactive plots, training history for all models studied, and detailed hyperparameter analysis—are available as an online report on Weights & Biases:

https://api.wandb.ai/links/mateo-personal/e6qxfrg6

#### 20-class classification

This variant extends the analysis to rotor degradation severity. The decision space maps to **20** unique classes:

- **Nominal (1 class):** No damage.
- **Single-rotor faults (8 classes):** Light (type 1) vs. heavy (type 2) damage for each of the four motors.
- **Multiple-rotor faults (11 classes):** Combinations of faults on two, three, or all four rotors simultaneously.

The task requires the model to estimate both **location** and **intensity** of damage.

**Brief result analysis:** Increasing the number of classes led to a slight drop in overall accuracy to **0.9773** (approx. **97.73%**). This is expected given lower separability between degradation levels on the same rotor. Despite a **fourfold** increase in classes (5 → 20), the performance drop is marginal (**< 1** percentage point), indicating robustness of the proposed architecture and sound input feature design.

### Appendix B: Source code

The research software was prepared and run in **Google Colab**, enabling GPU acceleration for neural network training. Two main Jupyter notebooks were developed:

1. **[FDI*UAV_W&B*(Colab).ipynb](FDI_UAV_W%26B_%28Colab%29.ipynb)** — Main script implementing the data pipeline, model definitions, and the training loop integrated with Weights & Biases experiment tracking.

2. **[FDI*UAV_Optuna*(Colab).ipynb](FDI_UAV_Optuna_%28Colab%29.ipynb)** — Script dedicated to hyperparameter optimization automation using the Optuna library.
