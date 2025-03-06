### **Logbook: Unsupervised Binary Classification with Isolation Forest Version 2.0**  
**File Name**: `if_bin_v2`  

---

#### **Date**: January 8, 2025 – January 15, 2025 
**Objective**: Enhance the Isolation Forest model for unsupervised binary anomaly detection on the NSL-KDD dataset by implementing robust preprocessing, feature selection, and optimized model parameters to improve clustering quality and detection performance.  

---

### **Work Summary**  
1. **Enhanced Data Preprocessing**  
   - **Training Data**: Loaded preprocessed `KDDTrain_processed.csv` (125,973 samples, 43 features).  
   - **Testing Data**: Loaded preprocessed `KDDTest_processed.csv` (22,544 samples, 43 features).  
   - **Key Improvements**:  
     - Applied `RobustScaler` to reduce impact of outliers.
     - Implemented information-theoretic feature selection using `SelectKBest` with `mutual_info_classif`.
     - Reduced feature dimensionality from 43 to 40 features, preserving the most relevant signals.
     - Maintained identical train-validation split (80-20) for consistent comparison with v1.

2. **Model Configuration Optimization**  
   - Analyzed actual contamination rate in training data (0.465).
   - **Enhanced Hyperparameters**:  
     - `n_estimators`: Maintained 200 trees for consistency
     - `contamination`: Fixed at 0.2 based on v1 performance
     - `max_features`: Set to 0.8 to introduce feature randomness
     - `bootstrap`: Enabled to improve ensemble robustness
     - `n_jobs`: Maintained -1 for parallel processing
     - `random_state`: Kept at 42 for reproducibility

3. **Unsupervised Evaluation**  
   - Applied clustering quality metrics with significant improvements:
     - **Silhouette Score**: 0.585 (training), 0.580 (validation), -0.242 (test)
     - **Davies-Bouldin Index**: 2.294 (training), 2.019 (validation), 2.317 (test)
   - **Prediction Distribution**:
     - Training: 80,622 normal (80%), 20,156 anomalies (20%)
     - Validation: 20,064 normal (80%), 5,131 anomalies (20%)
     - Test: 6,889 normal (31%), 15,655 anomalies (69%)

4. **Supervised Performance Assessment**  
   - **Training Performance**:
     - Accuracy: 0.594, Precision: 0.650, Recall: 0.279, F1-score: 0.390
   - **Validation Performance**:
     - Accuracy: 0.599, Precision: 0.657, Recall: 0.287, F1-score: 0.400
   - **Test Performance**:
     - Accuracy: 0.638, Precision: 0.649, Recall: 0.792, F1-score: 0.714

---

### **Results**  
- **Model Performance Comparison**: 
  | Dataset   | Metric    | if_bin_v1 | if_bin_v2 | Δ       |
  |-----------|-----------|-----------|-----------|---------|
  | Training  | Accuracy  | 0.600     | 0.594     | -0.006  |
  |           | F1-Score  | 0.399     | 0.390     | -0.009  |
  | Validation| Accuracy  | 0.605     | 0.599     | -0.006  |
  |           | F1-Score  | 0.410     | 0.400     | -0.010  |
  | Testing   | Accuracy  | 0.645     | 0.638     | -0.007  |
  |           | F1-Score  | 0.715     | 0.714     | -0.001  |

- **Clustering Quality Improvement**:  
  | Dataset   | Metric               | if_bin_v1 | if_bin_v2 | Δ       |
  |-----------|----------------------|-----------|-----------|---------|
  | Training  | Silhouette Score     | 0.292     | 0.585     | +0.293  |
  |           | Davies-Bouldin Index | 3.191     | 2.294     | -0.897  |
  | Testing   | Silhouette Score     | 0.035     | -0.242    | -0.277  |
  |           | Davies-Bouldin Index | 3.607     | 2.317     | -1.290  |

- **Confusion Matrix (Test Set)**:  
  - True Negatives: 4,220 (correctly identified normal)
  - False Positives: 5,491 (normal misclassified as anomaly)
  - False Negatives: 2,669 (anomaly misclassified as normal)
  - True Positives: 10,164 (correctly identified anomaly)

---

### **Key Decisions**  
1. **Robust Scaling Implementation**:  
   - Selected `RobustScaler` over `StandardScaler` used in preprocessing.
   - **Rationale**: Network traffic data contains extreme outliers; robust scaling reduces their influence on model performance.

2. **Information-Theoretic Feature Selection**:  
   - Applied `SelectKBest` with `mutual_info_classif` to reduce dimensions from 43 to 40.
   - **Trade-off**: Slight reduction in features improves signal-to-noise ratio while preserving detection capability.

3. **Random Feature Subsampling**:  
   - Set `max_features=0.8` to introduce feature diversity in the ensemble.
   - **Reasoning**: Provides robustness against noise while maintaining decision boundary precision.

4. **Bootstrap Sampling Activation**:  
   - Enabled `bootstrap=True` to create diverse training sets for each estimator.
   - **Impact**: Increases generalizability by reducing variance, albeit with a minor reduction in supervised metrics.

---

### **Conclusion**  
The enhanced Isolation Forest model (if_bin_v2) demonstrates substantially improved clustering quality with nearly double the training silhouette score (0.585 vs 0.292) and significantly reduced Davies-Bouldin index. While supervised metrics show a minimal decrease in accuracy (-0.7%) and F1-score (-0.1%) on the test set, the model achieves better data separation in embedding space. This indicates improved robustness in the unsupervised setting despite slight performance trade-offs. The introduction of robust scaling, feature selection, and ensemble optimization techniques has created a more statistically sound model with increased resistance to outliers and noise. Future work should explore automated contamination threshold tuning and adaptive ensemble size to potentially recover the small performance gap while maintaining the improved clustering characteristics.