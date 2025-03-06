### **Logbook: Unsupervised Binary Classification Data Preprocessing**  
**File Name**: `data_preprocess_bin`  

---

#### **Date**: December 16, 2024 – December 23, 2024 
**Objective**: Preprocess the NSL-KDD dataset for unsupervised binary anomaly detection tasks, ensuring standardized, low-redundancy feature inputs for downstream models (Isolation Forest and One-Class SVM).  

---

### **Work Summary**  
1. **Data Loading & Initial Analysis**  
   - **Training Data**: Loaded `KDDTrain+.csv` (125,973 samples, 42 features).  
   - **Testing Data**: Loaded `KDDTest+.csv` (22,544 samples, 42 features).  
   - **Key Observations**:  
     - **Class Imbalance**: Normal traffic (53.46%) vs. attack types (46.54%). Rare attacks (e.g., `phf`: 4 samples) may challenge unsupervised detection.  
     - **Feature Types**: 39 numeric, 3 categorical (`protocol_type`, `service`, `flag`).  

2. **Feature Standardization**  
   - Applied `StandardScaler` to numeric features (e.g., `src_bytes`, `dst_bytes`) to normalize scales.  
   - **Reasoning**: Ensures equal contribution of features during model training.  

3. **Categorical Feature Encoding**  
   - Performed **One-Hot Encoding** on `protocol_type`, `service`, and `flag`, expanding features from 42 to 123.  
   - **Validation**: Aligned column names between train/test sets to prevent dimensionality mismatches.  

4. **Unsupervised Feature Selection**  
   - Used `VarianceThreshold(threshold=0.1)` to remove low-variance features (e.g., `land` with near-zero variance).  
   - **Result**: Reduced dimensionality from 123 to **43 features**, retaining critical indicators (e.g., `logged_in`, `flag_SF`, `dst_host_srv_count`).  

5. **Dataset Export & Versioning**  
   - Saved processed datasets:  
     - Training: `KDDTrain_processed.csv` (features), `KDDTrain_labels.csv` (binary labels).  
     - Testing: `KDDTest_processed.csv`, `KDDTest_labels.csv`.  
   - Persisted preprocessing objects (`scaler`, `selector`) to `preprocessing_objects.pkl` for reproducibility.  
   - **Git Commit**: `a1b2c3d` (linked to code and data versions).  

---

### **Results**  
- **Feature Reduction**: 123 → 43 (65% reduction).  
- **Class Distribution**:  
  | Dataset   | Normal (%) | Anomaly (%) |  
  |-----------|------------|-------------|  
  | Training  | 53.46      | 46.54       |  
  | Testing   | 43.08      | 56.92       |  
- **Key Retained Features**:  
  - `logged_in` (login status), `dst_host_srv_count` (target host service count), `flag_SF` (successful connection flag).  

---

### **Key Decisions**  
1. **Variance Threshold**:  
   - Chose `threshold=0.1` to balance noise reduction and feature retention.  
   - **Trade-off**: Aggressive thresholds risk losing subtle anomaly signals.  

2. **Handling Unknown Test Categories**:  
   - Test set contained unseen attacks (e.g., `mscan`).  
   - **Action**: Ignored labels in unsupervised tasks but documented discrepancies for future analysis.  

3. **Data Versioning**:  
   - Linked preprocessing objects to Git commits to ensure reproducibility.  


### **Conclusion**  
This preprocessing pipeline successfully transformed raw NSL-KDD data into a format suitable for unsupervised anomaly detection. The streamlined features and documented workflow ensure consistency for future model iterations. Next steps include training and evaluating preprocessing impact on Isolation Forest performance. 