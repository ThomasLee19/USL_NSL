### **Logbook: Unsupervised Multiclass Classification with Isolation Forest Version 1.0**  
**File Name**: `if_multi_v1`  

---

#### **Date**: February 11, 2025 – February 13, 2025 
**Objective**: Implement a multiclass anomaly detection system using an ensemble of Isolation Forest models to classify network traffic into five distinct categories: Normal Traffic and four attack types (DOS, Probe, R2L, and U2R) on the NSL-KDD dataset.  

---

### **Work Summary**  
1. **Data Loading & Class Distribution Analysis**  
   - **Training Data**: Loaded preprocessed `KDDTrain_processed.csv` with multiclass labels (125,973 samples, 43 features).  
   - **Class Distribution**:  
     - Class 0 (Normal Traffic): 67,343 samples (53.46%)
     - Class 1 (DOS - Denial of Service): 45,927 samples (36.46%)
     - Class 2 (Probe - Surveillance/Scanning): 11,656 samples (9.25%)
     - Class 3 (R2L - Remote to Local): 995 samples (0.79%)
     - Class 4 (U2R - User to Root): 52 samples (0.04%)
   - **Data Split**: Maintained 80-20 training-validation split with stratification to preserve class distributions.

2. **Multi-model Architecture Implementation**  
   - **Model Design**: Implemented one Isolation Forest model per class (5 models total).
   - **Class-Specific Training**:
     - **Normal Class Model**: Trained exclusively on normal traffic samples.
     - **Attack Class Models**: Each trained on a balanced subset of normal traffic and specific attack samples.
   - **Sample Balance Strategy**:
     - For large attack classes (DOS, Probe): Used 5x more normal samples than attack samples.
     - For small attack classes (R2L, U2R): Used at least 1,000 normal samples to ensure robust boundary learning.

3. **Model Configuration & Hyperparameters**  
   - **Base Architecture**: All models used 200 trees with auto sample size.
   - **Contamination Parameters**: Class-specific contamination rates:
     - Normal Traffic: 0.1 (expecting 10% anomalies)
     - Attack Classes: 0.5 (balanced boundary between normal and attack)
   - **Training Samples Distribution**:
     - Class 0: 53,874 normal samples
     - Class 1: 53,874 normal + 36,741 DOS samples
     - Class 2: 46,625 normal + 9,325 Probe samples
     - Class 3: 3,980 normal + 796 R2L samples
     - Class 4: 1,000 normal + 42 U2R samples

4. **Classification Framework Implementation**  
   - **Anomaly Scoring**: Calculated anomaly scores from each model for each sample.
   - **Class Assignment**: Assigned class label based on the lowest anomaly score among all models.
   - **Evaluation Framework**: Used conventional classification metrics (accuracy, precision, recall, F1) for multiclass evaluation.

---

### **Results**  
- **Overall Performance**: 
  | Dataset    | Accuracy | Macro-Precision | Macro-Recall | Macro-F1 |
  |------------|----------|-----------------|--------------|----------|
  | Training   | 0.431    | 0.552           | 0.579        | 0.377    |
  | Validation | 0.428    | 0.553           | 0.546        | 0.376    |
  | Testing    | 0.399    | 0.486           | 0.374        | 0.306    |

- **Class-Specific Performance (Test Set)**:
  | Class | Description                  | Precision | Recall | F1-Score |
  |-------|------------------------------|-----------|--------|----------|
  | 0     | Normal Traffic               | 1.000     | 0.024  | 0.047    |
  | 1     | DOS (Denial of Service)      | 0.630     | 0.850  | 0.723    |
  | 2     | Probe (Surveillance/Scanning)| 0.660     | 0.460  | 0.543    |
  | 3     | R2L (Remote to Local)        | 0.135     | 0.448  | 0.208    |
  | 4     | U2R (User to Root)           | 0.006     | 0.090  | 0.012    |

- **Confusion Matrix (Test Set)**:  
  ```
  [[ 237 1185  320 7160  809]
   [   1 6338   74 1001   44]
   [   0 1207 1112   93    9]
   [   0 1309  164 1293  121]
   [   0   12   14   35    6]]
  ```
  - DOS attacks (class 1) showed the strongest classification performance
  - Normal traffic (class 0) was often misclassified as R2L (class 3)
  - U2R attacks (class 4) had the poorest detection performance

- **Class Distribution Shift (Test vs. Training)**:
  | Class | Training % | Test % | Δ % |
  |-------|------------|--------|-----|
  | 0     | 53.46      | 43.08  | -10.38 |
  | 1     | 36.46      | 33.08  | -3.38 |
  | 2     | 9.25       | 10.74  | +1.49 |
  | 3     | 0.79       | 12.81  | +12.02 |
  | 4     | 0.04       | 0.30   | +0.26 |
  
  - The significant increase in R2L attacks in the test set (from 0.79% to 12.81%) suggests the presence of previously unseen attack patterns

---

### **Key Decisions**  
1. **Multi-Model Architecture Selection**:  
   - Chose one-vs-rest approach with separate models instead of a single multiclass model.
   - **Rationale**: Individual models better capture the unique characteristics of each attack type and avoid domination by majority classes.
   - **Trade-off**: Increased computational requirements but allowed for class-specific contamination parameters.

2. **Adaptive Sample Balancing Strategy**:  
   - Used class-specific sampling ratios for model training.
   - **Reasoning**: Ensured sufficient examples of both normal and attack traffic while preventing normal class domination.
   - **Impact**: Improved detection rates for underrepresented attack classes at the expense of potential overfitting to limited samples.

3. **Asymmetric Contamination Parameters**:  
   - Applied lower contamination for normal traffic (0.1) compared to attack classes (0.5).
   - **Justification**: Normal traffic has lower expected anomaly rate than attack traffic, which requires a more balanced decision boundary.
   - **Result**: Improved model sensitivity for the majority of attack types, though with reduced performance for normal traffic detection.

4. **Score-Based Classification Approach**:  
   - Assigned final class based on minimizing anomaly scores across all models.
   - **Reasoning**: Leverages the strength of each model in recognizing its specific class patterns.
   - **Limitation**: Creates potential classification bias toward models that generally produce lower anomaly scores.

---

### **Conclusion**  
The multiclass Isolation Forest approach (if_multi_v1) demonstrates promising results for DOS and Probe attacks but struggles significantly with normal traffic classification (recall: 0.024) and rare attack types like U2R (F1: 0.012). The overall accuracy of 0.399 on the test set highlights substantial room for improvement. Future work should focus on the following priority areas:

1. **Contamination Parameter Tuning**:
   - Increase the contamination parameter for normal traffic (from 0.1 to 0.3-0.4) to improve recall.
   - Decrease the U2R contamination parameter (from 0.5 to 0.2-0.3) to reduce false positives.
   - Fine-tune parameters for other classes based on validation performance.

2. **Training Sample Balancing**:
   - Apply SMOTE or ADASYN oversampling for severely underrepresented classes (U2R and R2L).
   - Adjust normal-to-attack sample ratios, particularly for U2R (using 1:1 or 2:1 instead of current ratio).
   - Consider undersampling majority classes like DOS to create more balanced class distributions.

3. **Hierarchical Detection Framework**:
   - Implement a two-stage detection approach: first distinguishing normal vs. attack traffic, then classifying attack types.
   - This could address the poor normal traffic detection while simplifying the multiclass problem.

4. **Model Parameter Optimization**:
   - Explore different n_estimators values beyond the current 200 trees setting.
   - Test various max_samples configurations to improve model robustness.
   - Consider ensemble methods combining multiple anomaly detection algorithms.

5. **Feature Engineering**:
   - Analyze feature importance for each attack class and optimize feature selection.
   - Apply dimensionality reduction techniques to enhance class separation.
   - Create composite features that better characterize different attack patterns.

By prioritizing these improvements, particularly contamination parameter tuning and sample balancing which can be implemented with minimal modifications, the model's performance can be significantly enhanced for practical network intrusion detection systems.