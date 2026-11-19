[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MrEGAMERZ/netguard-ai/blob/main/NetGuard_AI_Intrusion_Detection.ipynb)

# NetGuard AI: Machine Learning Based Network Intrusion Detection System

NetGuard AI is a machine learning project that analyzes recorded network flow data to distinguish between normal, legitimate network traffic and malicious cyber attacks. 

The project is built as a self-contained Google Colab notebook. It trains and compares two different classification models: a classical Random Forest classifier and a Multi Layer Perceptron deep neural network.

---

## Quick Start in Google Colab

You can run the complete project directly in your browser without installing anything on your computer:

1. Click the **Open In Colab** badge at the top of this page.
2. In Google Colab, select **Runtime** from the top menu, then click **Change runtime type**.
3. Choose **T4 GPU** (Graphics Processing Unit) under Hardware accelerator and click **Save**.
4. Go to **Runtime** and click **Run all**.

All dependencies will install automatically, the dataset will stream in, and both models will train and display their evaluation metrics.

---

## Project Overview

Modern computer networks face continuous threats, ranging from automated port scans and brute force password attempts to large scale Denial of Service attacks. A Network Intrusion Detection System monitors network activity to identify unauthorized access or malicious behavior.

Traditional intrusion detection systems rely on fixed signatures (matching known patterns). While effective against previously seen attacks, signature based systems often fail against modified or zero day attacks. Machine learning provides an alternative approach by learning the statistical patterns that characterize attack traffic versus normal traffic.

NetGuard AI implements a binary classification workflow:
* **Benign**: Normal, everyday network communications.
* **Attack**: Malicious traffic, including Denial of Service, brute force attempts, port scans, and web attacks.

---

## Dataset Description

This project uses the **Canadian Institute for Cybersecurity Intrusion Detection System 2017** benchmark dataset, commonly abbreviated as **CIC-IDS2017**.

* **Origin**: Published by the University of New Brunswick and the Canadian Institute for Cybersecurity.
* **Traffic Nature**: Captures realistic network traffic generated in a controlled testbed environment over five consecutive days.
* **Record Type**: Rather than raw packet payloads, each record represents a bidirectional network flow summary (statistical aggregations calculated using the CICFlowMeter tool).
* **Sample Size**: We stream a representative sample of 200,000 records from a hosted copy on Hugging Face (`muzom/CIC-IDS2017`). This provides a strong statistical sample while keeping memory usage and training times fast inside Google Colab.

### Key Flow Features
Each network flow contains statistical measurements, such as:
* **Flow Duration**: Total time the connection remained open (in microseconds).
* **Total Forward and Backward Packets**: Number of packets sent in each direction.
* **Flow Bytes per Second**: Transmission rate of data.
* **Flow Packets per Second**: Transmission rate of packets.
* **Inter Arrival Time (IAT)**: Time elapsed between consecutive packets.
* **Packet Length Statistics**: Minimum, maximum, mean, and standard deviation of packet sizes.
* **Header Flags**: Counts of TCP (Transmission Control Protocol) flags such as SYN (Synchronize), ACK (Acknowledge), PSH (Push), and RST (Reset).

---

## Data Cleaning and Preprocessing

Real world network data often contains computational artifacts and irregular values. NetGuard AI applies the following preprocessing pipeline:

1. **Binary Label Conversion**: The original dataset includes over 14 fine grained attack classes (such as DoS Hulk, DoS GoldenEye, FTP Patator, and SSH Patator). We consolidate all attack types into a single label (`Attack = 1`) and keep benign traffic as (`Benign = 0`). This ensures a focused, clear binary classification problem.
2. **Infinite Value Handling**: Division by zero during flow rate calculations produces infinite numbers (`+inf`, `-inf`) in columns like Flow Bytes per Second. We replace all infinite values with the median of that respective column.
3. **Missing Value Imputation**: Any remaining missing or null values are filled using median imputation.
4. **Feature Selection**: We filter out non predictive columns (such as flow identifiers and timestamps) and retain 24 informative numeric flow metrics.
5. **Stratified Splitting**: The cleaned data is partitioned into 80 percent for training and 20 percent for testing. We use stratified sampling to maintain the exact ratio of benign to attack samples across both splits.
6. **Feature Normalization**: Features are standardized using scikit-learn's Standard Scaler (mean of zero and unit variance). Crucially, the scaler is fitted solely on the training partition and then applied to the test partition, preventing any data leakage.

---

## Machine Learning Models

We implement and contrast two complementary modeling paradigms:

### 1. Random Forest Classifier (Baseline Model)
* **Type**: Ensemble of decision trees trained with bagging (bootstrap aggregating).
* **Configuration**: 100 decision trees, maximum tree depth of 20, minimum samples per split of 5.
* **Hardware**: Runs in parallel on the Central Processing Unit using all available processor cores (`n_jobs = -1`).
* **Strengths**: Robust against outliers, resistant to overfitting, naturally provides feature importance scores, and requires minimal hyperparameter tuning.

### 2. Multi Layer Perceptron Neural Network (Deep Learning Model)
* **Type**: Feedforward deep artificial neural network.
* **Architecture**:
  * Input layer: 24 normalized flow features
  * Dense layer 1: 128 neurons, Rectified Linear Unit (ReLU) activation
  * Batch Normalization layer
  * Dropout layer (rate: 0.30)
  * Dense layer 2: 64 neurons, Rectified Linear Unit (ReLU) activation
  * Batch Normalization layer
  * Dropout layer (rate: 0.20)
  * Dense layer 3: 32 neurons, Rectified Linear Unit (ReLU) activation
  * Output layer: 1 neuron, Sigmoid activation function
* **Optimization**: Adam (Adaptive Moment Estimation) optimizer with a learning rate of 0.001.
* **Loss Function**: Binary Cross Entropy loss.
* **Callbacks**: Early Stopping with validation loss monitoring (restoring best weights) and Reduce Learning Rate on Plateau.

---

## Evaluation Metrics

Network intrusion detection systems operate under class imbalance, meaning normal traffic often outnumbers attack traffic. Because of this, accuracy alone can be misleading. We evaluate both models across four standard metrics:

* **Accuracy**: The proportion of all predictions that were correct.
  $$\text{Accuracy} = \frac{\text{True Positives} + \text{True Negatives}}{\text{Total Records}}$$

* **Precision**: When the model sounds an alarm and flags a connection as an attack, how often is it actually an attack? High precision prevents alert fatigue for security analysts.
  $$\text{Precision} = \frac{\text{True Positives}}{\text{True Positives} + \text{False Positives}}$$

* **Recall (Sensitivity)**: Out of all the real attacks present in the data, how many did the system catch? In security, recall is typically the most critical metric because a missed breach is far more damaging than a false alarm.
  $$\text{Recall} = \frac{\text{True Positives}}{\text{True Positives} + \text{False Negatives}}$$

* **F1 Score**: The harmonic mean of precision and recall. It ensures balanced performance between detecting threats and avoiding false alarms.
  $$\text{F1 Score} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$

The notebook generates:
* Full classification reports for both models
* Confusion matrices displayed as clear visual heatmaps
* Feature importance bar charts from the Random Forest
* Training and validation loss and accuracy curves across epochs for the neural network
* Side by side comparative bar charts

---

## Interactive Prediction Demo

Stage 10 of the notebook provides a dedicated `predict_traffic()` function. 

You can call this function on any sample from the held out test set. It extracts the record, passes it through the fitted scaler, queries both the Random Forest and the Neural Network, and prints:
* The actual ground truth label
* The Random Forest prediction and its confidence percentage
* The Neural Network prediction and its sigmoid probability score
* An agreement assessment indicating whether both models agreed and whether they were correct

---

## Project Structure

```
netguard-ai/
├── NetGuard_AI_Intrusion_Detection.ipynb   # Complete, self-contained Google Colab notebook
├── README.md                              # Project documentation and guide
├── requirements.txt                       # Python dependencies for local execution
├── LICENSE                                # MIT Open Source License
└── .gitignore                             # Ignored cache and temporary files
```

---

## Academic Presentation and Viva Voce Guide

If you are presenting or defending this project in an academic evaluation or practical exam, here are clear explanations for the questions most commonly asked:

### 1. Why use network flow features instead of raw packet payloads?
Raw packet inspection (Deep Packet Inspection) requires decrypting traffic and reading packet contents. In modern networks, over 90 percent of traffic is encrypted using Transport Layer Security (TLS/HTTPS). Flow statistics (such as packet counts, timing intervals, and packet size distributions) do not require decryption, preserve user privacy, and demand significantly less computing power.

### 2. Why did we combine all attacks into a single binary label?
A real world triage system first answers a fundamental question: "Is this connection safe or dangerous?" Grouping specific attacks into an overall Attack class makes the classification problem well posed, prevents extreme class fragmentation, and produces straightforward, explainable metrics.

### 3. Why is Recall considered more important than Precision in cybersecurity?
A false positive (flagging normal traffic as an attack) results in an alert that an administrator can verify. A false negative (classifying an actual attack as benign) means an intruder has penetrated the network undetected. Therefore, maximizing Recall is the primary objective of most defense systems.

### 4. What is the difference between Random Forest and the Neural Network here?
Random Forest relies on an ensemble of independent decision trees that vote on the outcome. It is very fast to train on tabular flow numbers and provides direct feature importance rankings. The Multi Layer Perceptron neural network uses layers of mathematical transformations with non linear activation functions, optimized using gradient descent and backpropagation. Comparing both demonstrates classical machine learning against modern deep learning.

### 5. What are the key limitations of this system?
* **Offline analysis**: The system evaluates recorded flow records. It is not currently connected to a live packet capture driver (like libpcap) to inspect live network interfaces.
* **Dataset age**: The CIC-IDS2017 dataset was recorded in 2017. While fundamental attack patterns (such as Denial of Service floods and brute force scans) remain relevant, modern zero day exploits may show different flow signatures.
* **Adversarial evasion**: Attackers aware of flow based detection may deliberately insert delays or pad packet sizes to mimic legitimate browsing activity.

---

## Local Installation

If you prefer to run the notebook on your local computer using Jupyter:

```bash
# Clone the repository
git clone https://github.com/MrEGAMERZ/netguard-ai.git
cd netguard-ai

# Create a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install required packages
pip install -r requirements.txt

# Start Jupyter Notebook
jupyter notebook NetGuard_AI_Intrusion_Detection.ipynb
```

---

## Requirements

The project uses standard, reliable open source data science libraries:
* Python 3.9 or higher
* NumPy
* Pandas
* Matplotlib
* Seaborn
* scikit-learn
* TensorFlow
* datasets (Hugging Face)

---

## License

This project is licensed under the MIT License. You are free to use, modify, and distribute this software for educational and personal projects. See the [LICENSE](LICENSE) file for full details.
