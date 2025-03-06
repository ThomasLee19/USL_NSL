### **Logbook: Unsupervised Binary Classification with Isolation Forest Version 3.0**  
**File Name**: `if_bin_v3`  

---

#### **Date**: January 20, 2025 – January 27, 2025 
**Objective**: Optimize the Isolation Forest model through systematic hyperparameter tuning to maximize F1-score while balancing precision and recall for unsupervised anomaly detection on the NSL-KDD dataset.  

---

### **Work Summary**  
1. **Data Preprocessing Pipeline**  
   - **Training Data**: Maintained preprocessing approach from v2 with robust scaling.
   - **Feature Selection**: Continued using mutual information-based feature selection (40 features).
   - **Key Focus**: Ensured preprocessing consistency with v2 to isolate the impact of hyperparameter optimization.

2. **Systematic Hyperparameter Tuning**  
   - Implemented comprehensive grid search with cross-validation:
     - **Hyperparameter Space**: 162 combinations (3×3×3×3×2) across 5 parameters.
     - **Cross-Validation**: 3-fold stratified CV for robust parameter estimation.
     - **Custom Scorer**: Developed specialized F1-score function to handle isolation forest's prediction format.
   - **Optimized Parameters**:  
     - `n_estimators`: [100, 200, 300] → Best: 300
     - `max_samples`: [0.5, 0.8, 'auto'] → Best: 'auto'
     - `contamination`: [0.1, 0.2, 0.3] → Best: 0.3
     - `max_features`: [0.6, 0.8, 1.0] → Best: 0.6
     - `bootstrap`: [True, False] → Best: True
   - **Best CV Score**: F1-score of 0.641 during cross-validation.

3. **Model Training & Evaluation Framework**  
   - Retrained final model using best parameters on full training set.
   - Implemented comprehensive evaluation metrics:
     - Standard classification metrics (accuracy, precision, recall, F1)
     - Clustering quality metrics (silhouette score, Davies-Bouldin index)
     - Added framework for ROC analysis and visualization.

4. **Performance Analysis**  
   - **Training Performance**:
     - Accuracy: 0.656, Precision: 0.702, Recall: 0.453, F1-score: 0.550
     - Silhouette Score: 0.384, Davies-Bouldin Index: 2.398
   - **Validation Performance**:
     - Accuracy: 0.662, Precision: 0.710, Recall: 0.462, F1-score: 0.559
   - **Test Performance**:
     - Accuracy: 0.521, Precision: 0.549, Recall: 0.892, F1-score: 0.680
     - Silhouette Score: -0.649, Davies-Bouldin Index: 1.992

---

### **Results**  
- **Model Performance Comparison Across Versions**: 
  | Dataset   | Metric    | if_bin_v1 | if_bin_v2 | if_bin_v3 | Δ (v3-v2) |
  |-----------|-----------|-----------|-----------|-----------|-----------|
  | Training  | F1-Score  | 0.399     | 0.390     | 0.550     | +0.160    |
  | Validation| F1-Score  | 0.410     | 0.400     | 0.559     | +0.159    |
  | Testing   | F1-Score  | 0.715     | 0.714     | 0.680     | -0.034    |
  | Testing   | Recall    | 0.784     | 0.792     | 0.892     | +0.100    |
  | Testing   | Precision | 0.658     | 0.649     | 0.549     | -0.100    |

- **Clustering Quality Improvement**:  
  | Dataset   | Metric               | if_bin_v1 | if_bin_v2 | if_bin_v3 | Δ (v3-v2) |
  |-----------|----------------------|-----------|-----------|-----------|-----------|
  | Training  | Silhouette Score     | 0.292     | 0.585     | 0.384     | -0.201    |
  |           | Davies-Bouldin Index | 3.191     | 2.294     | 2.398     | +0.104    |
  | Testing   | Silhouette Score     | 0.035     | -0.242    | -0.649    | -0.407    |
  |           | Davies-Bouldin Index | 3.607     | 2.317     | 1.992     | -0.325    |

- **Prediction Distribution**:  
  | Dataset   | if_bin_v2          | if_bin_v3          | Δ % Anomaly |
  |-----------|--------------------|--------------------|-------------|
  | Training  | 80,622:20,156 (20%)| 70,544:30,234 (30%)| +10%        |
  | Validation| 20,064:5,131 (20%) | 17,564:7,631 (30%) | +10%        |
  | Testing   | 6,889:15,655 (69%) | 1,694:20,850 (92%) | +23%        |

- **Confusion Matrix (Test Set)**:  
  - True Negatives: 307 (correctly identified normal)
  - False Positives: 9,404 (normal misclassified as anomaly)
  - False Negatives: 1,387 (anomaly misclassified as normal)
  - True Positives: 11,446 (correctly identified anomaly)

---

### **Key Decisions**  
1. **Contamination Parameter Increase**:  
   - Increased from 0.2 to 0.3 based on grid search results.
   - **Trade-off**: Higher recall (0.892) at cost of precision (0.549) and accuracy (0.521).
   - **Justification**: In security contexts, missing attacks (false negatives) is more costly than false alarms.

2. **Feature Randomness Enhancement**:  
   - Reduced `max_features` from 0.8 to 0.6.
   - **Rationale**: Increased diversity in tree construction improves ensemble robustness.
   - **Impact**: Contributed to better training/validation performance but created more generalization challenges on test data.

3. **Ensemble Size Expansion**:  
   - Increased `n_estimators` from 200 to 300.
   - **Reasoning**: Larger ensemble reduces variance but increases computational cost.
   - **Evidence**: Improved cross-validation F1-score from previous versions.

4. **Custom F1 Scorer Implementation**:  
   - Developed specialized scoring function for grid search to handle potential numeric issues.
   - **Technical detail**: Implemented nan/infinity handling and value clipping for robust optimization.

---

### **Conclusion**  
Version 3 of the Isolation Forest model (if_bin_v3) demonstrates the trade-offs inherent in anomaly detection through systematic hyperparameter optimization. The model achieves significantly improved training and validation F1-scores (+0.16) through grid search optimization, indicating better parameter fit for the development dataset. However, test set performance shows a slight decrease in F1-score (-0.034) despite substantial gains in recall (+0.10), revealing a precision-recall trade-off common in anomaly detection. The significant shift in prediction distribution (92% anomalies on test set) suggests a more aggressive anomaly flagging approach that prioritizes attack detection over false alarm reduction. This model would be particularly appropriate for high-security environments where missing an attack carries greater risk than investigating false positives. Future work should explore cost-sensitive learning approaches to better balance the precision-recall trade-off based on operational requirements.