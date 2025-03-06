### **Logbook: Unsupervised Binary Classification with Isolation Forest Version 1.0**  
**File Name**: `if_bin_v1`  

---

#### **Date**: December 24, 2024 – December 25, 2024; January 2, 2025 - January 6, 2025
**Objective**: Train and evaluate an Isolation Forest model for unsupervised binary anomaly detection on the preprocessed NSL-KDD dataset, assessing performance using both unsupervised clustering metrics and supervised evaluation against ground-truth labels.  

---

### **Work Summary**  
1. **Data Loading & Preparation**  
   - **Training Data**: Loaded preprocessed `KDDTrain_processed.csv` (125,973 samples, 43 features).  
   - **Testing Data**: Loaded preprocessed `KDDTest_processed.csv` (22,544 samples, 43 features).  
   - **Key Actions**:  
     - Split data into train (80%, 100,778 samples) and validation (20%, 25,195 samples) sets, maintaining class distribution.
     - Confirmed feature alignment between datasets.

2. **Model Configuration & Training**  
   - Implemented Isolation Forest with GPU acceleration via CUDF and CuPy.
   - **Hyperparameters**:  
     - `n_estimators`: 200 trees (for robust anomaly detection)
     - `contamination`: 0.2 (expected anomaly ratio)
     - `max_samples`: 'auto' (uses default heuristic) 
     - `random_state`: 42 (for reproducibility)
     - `n_jobs`: -1 (parallel processing on all CPU cores)

3. **Unsupervised Evaluation**  
   - Applied clustering quality metrics:
     - **Silhouette Score**: 0.292 (training), 0.035 (test)
     - **Davies-Bouldin Index**: 3.191 (training), 3.607 (test)
   - **Prediction Distribution**:
     - Training: 80,622 normal (80%), 20,156 anomalies (20%)
     - Validation: 20,057 normal (80%), 5,138 anomalies (20%)
     - Test: 7,241 normal (32%), 15,303 anomalies (68%)

4. **Supervised Performance Assessment**  
   - Converted isolation forest predictions to binary labels for comparison with ground truth.
   - **Training Performance**:
     - Accuracy: 0.600, Precision: 0.664, Recall: 0.285, F1-score: 0.399
   - **Validation Performance**:
     - Accuracy: 0.605, Precision: 0.673, Recall: 0.295, F1-score: 0.410
   - **Test Performance**:
     - Accuracy: 0.645, Precision: 0.658, Recall: 0.784, F1-score: 0.715

---

### **Results**  
- **Model Performance**: 
  | Dataset   | Accuracy | Precision | Recall | F1-Score |  
  |-----------|----------|-----------|--------|----------|  
  | Training  | 0.600    | 0.664     | 0.285  | 0.399    |  
  | Validation| 0.605    | 0.673     | 0.295  | 0.410    |  
  | Testing   | 0.645    | 0.658     | 0.784  | 0.715    |  

- **Clustering Quality**:  
  | Dataset   | Silhouette Score | Davies-Bouldin Index |  
  |-----------|------------------|----------------------|  
  | Training  | 0.292            | 3.191                |  
  | Testing   | 0.035            | 3.607                |  

- **Confusion Matrix (Test Set)**:  
  - True Negatives: 4,473 (correctly identified normal)
  - False Positives: 5,238 (normal misclassified as anomaly)
  - False Negatives: 2,768 (anomaly misclassified as normal)
  - True Positives: 10,065 (correctly identified anomaly)

---

### **Key Decisions**  
1. **Contamination Parameter**:  
   - Chose `contamination=0.2` based on approximate anomaly ratio in the training set.  
   - **Trade-off**: Higher values increase anomaly detection sensitivity but risk more false positives.

2. **Ensemble Size**:  
   - Selected 200 estimators for the Isolation Forest.
   - **Reasoning**: Balanced computational efficiency with ensemble robustness.

3. **Evaluation Approach**:  
   - Employed both unsupervised and supervised metrics to provide comprehensive evaluation.
   - **Rationale**: Unsupervised metrics evaluate clustering quality, while supervised metrics assess real-world detection performance.

---

### **Conclusion**  
The Isolation Forest model (if_bin_v1) demonstrated moderate performance for unsupervised anomaly detection on the NSL-KDD dataset. While training and validation metrics showed consistent behavior, the model performed significantly better on the test set (F1-score: 0.715) despite lower clustering quality metrics. This discrepancy suggests the model captures useful anomaly patterns despite suboptimal data separation. The higher recall (0.784) on test data indicates promising capability in identifying true anomalies, though the precision (0.658) shows room for improvement in reducing false positives. Future work should explore alternative contamination values and feature selection techniques to enhance detection performance.