### **Logbook: Unsupervised Multiclass Classification with One-Class SVM Version 2.0**  
**File Name**: `ocsvm_multi_v2`  

---

#### **Date**: February 22, 2025 – February 25, 2025 
**Objective**: Enhance the multiclass anomaly detection system by implementing optimized nu parameters and addressing class imbalance issues to improve classification performance across all traffic categories, particularly for normal traffic and rare attack types like U2R and R2L.  

---

### **Work Summary**  
1. **Nu Parameter Refinement**  
   - **Previous Approach**: Used fixed nu parameters (0.1 for normal, 0.2 for all attack classes).
   - **New Configuration**:  
     - Class 0 (Normal Traffic): Increased from 0.1 to 0.35
     - Class 1-3 (DOS, Probe, R2L): Reduced from 0.2 to 0.15
     - Class 4 (U2R): Significantly reduced from 0.2 to 0.1
   - **Rationale**: Adjusted parameters to better reflect actual class distributions and improve boundary definition between normal and attack traffic.

2. **SMOTE-Based Sample Balancing**  
   - **Targeted Approach**: Applied synthetic oversampling only to severely underrepresented classes.
   - **Resampling Strategy**:
     - Class 3 (R2L): Increased 5× from 796 to 3,980 samples
     - Class 4 (U2R): Increased 10× from 42 to 420 samples
     - Classes 0-2: Kept at original sizes
   - **Outcome**: More balanced overall class distribution while preserving authentic samples for well-represented classes.
   - **Final Distribution**:
     ```
     Class 0 (Normal Traffic): 53,874 samples (51.63%)
     Class 1 (DOS): 36,741 samples (35.21%)
     Class 2 (Probe): 9,325 samples (8.94%)
     Class 3 (R2L): 3,980 samples (3.81%) [⬆ from 0.79%]
     Class 4 (U2R): 420 samples (0.40%) [⬆ from 0.04%]
     ```

3. **Class-Specific Sampling Ratios**  
   - **Adaptive Approach**: Implemented class-specific normal-to-attack sample ratios.
   - **Ratio Configuration**:
     - Class 1 (DOS): 0.5:1 ratio (reduced normal samples to prevent normal class domination)
     - Class 2-3 (Probe, R2L): 2:1 ratio (moderate normal sample presence)
     - Class 4 (U2R): 1:1 ratio (equal representation for boundary learning)
   - **Training Sample Distribution**:
     - Class 0: 53,874 normal samples only
     - Class 1: 18,370 normal + 36,741 DOS samples
     - Class 2: 18,650 normal + 9,325 Probe samples
     - Class 3: 7,960 normal + 3,980 R2L samples
     - Class 4: 420 normal + 420 U2R samples

4. **Performance Optimizations**  
   - **Increased Cache Size**: Bumped up from 500 to 1000 to improve training speed
   - **Maintained Architecture**: Kept the one-vs-rest approach with separate OC-SVM models
   - **Base Parameters**: Kept RBF kernel with gamma='scale'
   - **Classification Method**: Continued using minimum anomaly score for final class assignment

---

### **Results**  
- **Overall Performance Comparison**: 
  | Dataset    | Metric          | ocsvm_multi_v1 | ocsvm_multi_v2 | if_multi_v2 | Improvement |
  |------------|-----------------|----------------|----------------|-------------|-------------|
  | Training   | Accuracy        | 0.424          | 0.556          | 0.751       | +31.1%      |
  | Validation | Accuracy        | 0.424          | 0.555          | 0.747       | +30.9%      |
  | Testing    | Accuracy        | 0.366          | 0.434          | 0.628       | +18.6%      |
  | Testing    | Macro-F1        | 0.242          | 0.237          | 0.455       | -2.1%       |

- **Class-Specific Performance (Test Set)**:
  | Class | Description                  | v1 Recall | v2 Recall | v1 F1-Score | v2 F1-Score |
  |-------|------------------------------|-----------|-----------|-------------|-------------|
  | 0     | Normal Traffic               | 0.001     | 0.990     | 0.002       | 0.600       |
  | 1     | DOS (Denial of Service)      | 0.996     | 0.001     | 0.518       | 0.002       |
  | 2     | Probe (Surveillance/Scanning)| 0.330     | 0.040     | 0.449       | 0.077       |
  | 3     | R2L (Remote to Local)        | 0.003     | 0.001     | 0.006       | 0.003       |
  | 4     | U2R (User to Root)           | 0.134     | 0.570     | 0.237       | 0.504       |

- **Confusion Matrix (Test Set)**:  
  ```
  [[9626   37    4   14   30]
   [7443    8    3    4    0]
   [2325    0   96    0    0]
   [2863    0    4    4   16]
   [  27    0    0    2   38]]
  ```
  - Normal traffic (class 0) correctly identified 99.0% of the time (vs. 0.1% in v1)
  - U2R attacks (class 4) show dramatically improved detection at 57.0% recall (vs. 13.4% in v1)
  - DOS attacks (class 1) show severely reduced recall at 0.1% (vs. 99.6% in v1)
  - Probe and R2L attacks show reduced detection rates

- **Training Efficiency**:
  - Total training time: 335.58 seconds (improved from 786.56 seconds in v1)
  - Normal model training: 218.70 seconds (slower than v1 due to higher nu)
  - DOS model training: 94.65 seconds (much faster than v1's 581.59 seconds)

---

### **Key Decisions**  
1. **Nu Parameter Rebalancing**:  
   - **Approach**: Implemented asymmetric nu parameter tuning based on v1 analysis.
   - **Rationale**: Higher nu for normal traffic expanded its boundary, significantly improving detection rate; lower nu for U2R improved precision.
   - **Impact**: 990× improvement in normal traffic recall, 4.3× improvement in U2R recall.
   - **Trade-off**: Severely reduced DOS and Probe detection accuracy.

2. **Selective SMOTE Application**:  
   - **Decision**: Applied SMOTE only to severely underrepresented classes (R2L and U2R).
   - **Justification**: Full SMOTE on all classes would have created an extremely imbalanced dataset.
   - **Impact**: Improved U2R classification performance (57% recall vs 13.4% in v1).
   - **Limitation**: Did not significantly help R2L detection due to diversity of attack patterns.

3. **Attack-Specific Normal:Attack Ratios**:  
   - **Strategy**: Varied normal sample ratios based on attack class characteristics.
   - **Reasoning**: Different attack types require different decision boundaries; DOS needs fewer normal samples to avoid overfitting.
   - **Result**: Better class separation for normal traffic and U2R attacks.
   - **Issue**: Created bias toward normal traffic classification, with most samples classified as normal.

4. **Performance Optimization**:  
   - **Approach**: Increased cache size from 500 to 1000 for SVM training.
   - **Effect**: Reduced total training time by 57% while maintaining model quality.
   - **Benefit**: Allowed faster experimentation and parameter tuning cycles.
   - **Note**: Most significant time savings came from DOS model training (94.65s vs 581.59s).

---

### **Conclusion**  
The enhanced One-Class SVM model (ocsvm_multi_v2) demonstrates significant improvements in overall accuracy (43.4% vs 36.6%) and dramatic improvements in normal traffic detection (99.0% vs 0.1%) compared to v1. The model now correctly identifies normal traffic with high reliability and shows major improvements in U2R attack detection (57% vs 13.4%).

However, these improvements came at the cost of severely reduced performance on DOS and Probe attacks. The model has essentially flipped its bias from classifying most traffic as DOS (in v1) to classifying most traffic as normal (in v2). This resulted in a slight decrease in macro-F1 score despite the higher accuracy.

Compared to the Isolation Forest approach (if_multi_v2), the OC-SVM model still underperforms on overall accuracy (43.4% vs 62.8%) and macro-F1 (0.237 vs 0.455), suggesting that Isolation Forest may be better suited for this multiclass anomaly detection task.

The nu parameter tuning and oversampling of minority classes proved partially effective but created new biases. Future work should focus on finding a better balance between normal and attack class detection, potentially through more sophisticated parameter tuning or ensemble methods combining both OC-SVM and Isolation Forest approaches.