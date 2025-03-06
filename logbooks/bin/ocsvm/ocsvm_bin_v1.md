### **Logbook: Unsupervised Binary Classification with One-Class SVM Version 1.0**  
**File Name**: `ocsvm_bin_v1`  

---

#### **Date**: January 28, 2025 – February 3, 2025 
**Objective**: Implement and evaluate a One-Class SVM model for unsupervised binary anomaly detection on the NSL-KDD dataset, providing an alternative approach to the Isolation Forest method while maintaining comparable evaluation frameworks.  

---

### **Work Summary**  
1. **Data Loading & Preparation**  
   - **Training Data**: Loaded preprocessed `KDDTrain_processed.csv` (125,973 samples, 43 features).  
   - **Testing Data**: Loaded preprocessed `KDDTest_processed.csv` (22,544 samples, 43 features).  
   - **Key Actions**:  
     - Split data into train (80%, 100,778 samples) and validation (20%, 25,195 samples) sets.
     - Used raw preprocessed features without additional scaling or selection to establish baseline performance.

2. **Model Configuration & Training**  
   - Implemented One-Class SVM with scikit-learn's implementation.
   - **Hyperparameters**:  
     - `kernel`: 'rbf' (Radial Basis Function) for nonlinear decision boundary
     - `nu`: 0.2 (upper bound on training error fraction and lower bound on support vector fraction)
     - `gamma`: 'scale' (1 / (n_features * X.var())) for kernel coefficient
     - `cache_size`: 500 MB to optimize performance on large dataset

3. **Unsupervised Evaluation**  
   - Applied clustering quality metrics:
     - **Silhouette Score**: 0.324 (training), 0.274 (test)
     - **Davies-Bouldin Index**: 3.962 (training), 3.403 (test)
   - **Prediction Distribution**:
     - Training: 80,623 normal (80%), 20,155 anomalies (20%)
     - Validation: 19,965 normal (79%), 5,230 anomalies (21%)
     - Test: 15,426 normal (68%), 7,118 anomalies (32%)

4. **Supervised Performance Assessment**  
   - **Training Performance**:
     - Accuracy: 0.586, Precision: 0.627, Recall: 0.270, F1-score: 0.377
   - **Validation Performance**:
     - Accuracy: 0.584, Precision: 0.624, Recall: 0.277, F1-score: 0.384
   - **Test Performance**:
     - Accuracy: 0.646, Precision: 0.840, Recall: 0.466, F1-score: 0.600

---

### **Results**  
- **Model Performance Comparison with Isolation Forest**: 
  | Dataset   | Metric    | IF_bin_v1 | IF_bin_v2 | IF_bin_v3 | OCSVM_v1 |
  |-----------|-----------|-----------|-----------|-----------|----------|
  | Test      | Accuracy  | 0.645     | 0.638     | 0.521     | 0.646    |
  |           | Precision | 0.658     | 0.649     | 0.549     | 0.840    |
  |           | Recall    | 0.784     | 0.792     | 0.892     | 0.466    |
  |           | F1-Score  | 0.715     | 0.714     | 0.680     | 0.600    |

- **Clustering Quality**:  
  | Dataset   | Metric               | IF_bin_v1 | OCSVM_v1 | Difference |
  |-----------|----------------------|-----------|----------|------------|
  | Training  | Silhouette Score     | 0.292     | 0.324    | +0.032     |
  |           | Davies-Bouldin Index | 3.191     | 3.962    | +0.771     |
  | Testing   | Silhouette Score     | 0.035     | 0.274    | +0.239     |
  |           | Davies-Bouldin Index | 3.607     | 3.403    | -0.204     |

- **Prediction Distribution (Test Set)**:  
  | Model     | Normal | Anomaly | % Anomaly |
  |-----------|--------|---------|-----------|
  | IF_bin_v1 | 7,241  | 15,303  | 68%       |
  | IF_bin_v2 | 6,889  | 15,655  | 69%       |
  | IF_bin_v3 | 1,694  | 20,850  | 92%       |
  | OCSVM_v1  | 15,426 | 7,118   | 32%       |

- **Confusion Matrix (Test Set)**:  
  - True Negatives: 8,574 (correctly identified normal)
  - False Positives: 1,137 (normal misclassified as anomaly)
  - False Negatives: 6,852 (anomaly misclassified as normal)
  - True Positives: 5,981 (correctly identified anomaly)

---

### **Key Decisions**  
1. **Kernel Selection**:  
   - Chose `rbf` kernel over linear for detecting complex, non-linear decision boundaries.
   - **Rationale**: Network attacks often exhibit nonlinear patterns that RBF captures better than linear kernels.

2. **Nu Parameter Setting**:  
   - Set `nu=0.2` based on approximate anomaly ratio in the training set.
   - **Trade-off**: Lower values increase precision but reduce recall; higher values would increase false positives.

3. **Unmodified Feature Space**:  
   - Used original preprocessed features without additional transformations.
   - **Reasoning**: Establish baseline performance before implementing more complex feature engineering.
   - **Impact**: Potentially limits model performance but provides clear comparison point for future iterations.

4. **Memory Optimization**:  
   - Increased `cache_size` to 500MB to accelerate training on large dataset.
   - **Importance**: SVM training scales poorly with dataset size; memory optimization is critical for performance.

---

### **Conclusion**  
The One-Class SVM model (ocsvm_bin_v1) demonstrates a distinctly different approach to anomaly detection compared to Isolation Forest, achieving the highest precision (0.840) among all tested models at the cost of lower recall (0.466). This high-precision characteristic makes it particularly suitable for applications where false positive minimization is critical. The OCSVM model also shows better test set clustering quality with higher silhouette scores than IF_bin_v1, suggesting better-defined decision boundaries. However, the substantially lower anomaly prediction rate (32% vs. 68-92% in IF models) indicates a more conservative detection approach that misses a significant portion of anomalies. Future work should explore optimizing gamma and nu parameters, as well as implementing feature selection and scaling techniques that might enhance recall while maintaining the model's excellent precision.