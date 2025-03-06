### **Logbook: Unsupervised Multiclass Classification with One-Class SVM Version 1.0**  
**File Name**: `ocsvm_multi_v1`  

---

#### **Date**: February 18, 2025 – February 21, 2025 
**Objective**: Implement a multiclass anomaly detection system using an ensemble of One-Class SVM models to classify network traffic into five distinct categories: Normal Traffic and four attack types (DOS, Probe, R2L, and U2R) on the NSL-KDD dataset.  

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
   - **Data Split**: Used 80-20 training-validation split with stratification to keep class distributions the same.

2. **Multi-model Architecture Implementation**  
   - **Model Design**: Created one One-Class SVM model for each class (5 models total).
   - **Class-Specific Training**:
     - **Normal Class Model**: Trained only on normal traffic samples.
     - **Attack Class Models**: Each trained on a mix of normal traffic and specific attack samples.
   - **Sample Balance Strategy**:
     - For big attack classes (DOS, Probe): Used 5x more normal samples than attack samples.
     - For small attack classes (R2L, U2R): Used at least 1,000 normal samples to help the model learn better.

3. **Model Configuration & Hyperparameters**  
   - **Base Architecture**: All models used RBF kernel with gamma='scale'.
   - **Nu Parameters**: Different nu values for different classes:
     - Normal Traffic: 0.1 (expecting 10% anomalies)
     - Attack Classes: 0.2 (for balanced decision boundary)
   - **Training Samples Distribution**:
     - Class 0: 53,874 normal samples
     - Class 1: 53,874 normal + 36,741 DOS samples
     - Class 2: 46,625 normal + 9,325 Probe samples
     - Class 3: 3,980 normal + 796 R2L samples
     - Class 4: 1,000 normal + 42 U2R samples

4. **Classification Framework Implementation**  
   - **Anomaly Scoring**: Got anomaly scores from each model for all samples.
   - **Class Assignment**: Picked the class with the lowest anomaly score.
   - **Evaluation**: Used standard metrics (accuracy, precision, recall, F1) to evaluate performance.

---

### **Results**  
- **Overall Performance Comparison**: 
  | Dataset    | Metric          | if_multi_v1 | if_multi_v2 | ocsvm_multi_v1 | vs. if_multi_v1 |
  |------------|-----------------|-------------|-------------|----------------|-----------------|
  | Training   | Accuracy        | 0.431       | 0.751       | 0.424          | -1.6%           |
  | Validation | Accuracy        | 0.428       | 0.747       | 0.424          | -0.9%           |
  | Testing    | Accuracy        | 0.399       | 0.628       | 0.366          | -8.3%           |
  | Testing    | Macro-F1        | 0.306       | 0.455       | 0.242          | -20.9%          |

- **Class-Specific Performance (Test Set)**:
  | Class | Description                  | if_v1 Recall | if_v2 Recall | ocsvm Recall | if_v1 F1 | if_v2 F1 | ocsvm F1 |
  |-------|------------------------------|--------------|--------------|--------------|----------|----------|----------|
  | 0     | Normal Traffic               | 0.024        | 0.650        | 0.001        | 0.047    | 0.740    | 0.002    |
  | 1     | DOS (Denial of Service)      | 0.850        | 0.700        | 0.996        | 0.723    | 0.800    | 0.518    |
  | 2     | Probe (Surveillance/Scanning)| 0.460        | 0.870        | 0.330        | 0.543    | 0.530    | 0.449    |
  | 3     | R2L (Remote to Local)        | 0.448        | 0.160        | 0.003        | 0.208    | 0.160    | 0.006    |
  | 4     | U2R (User to Root)           | 0.090        | 0.400        | 0.134        | 0.012    | 0.050    | 0.237    |

- **Confusion Matrix (Test Set)**:  
  ```
  [[  10 9569  118   14    0]
   [   0 7424   34    0    0]
   [   0 1623  798    0    0]
   [   3 2698  177    9    0]
   [   1   26   19   12    9]]
  ```
  - DOS attacks (class 1) had the best detection rate (99.6%)
  - Almost all normal traffic was wrongly classified as DOS
  - U2R attacks (class 4) had perfect precision but detected only 13.4% of cases
  - R2L attacks (class 3) were barely detected at all

- **Training Time**:
  - Total training time: 786.56 seconds
  - DOS model took the longest (581.59 seconds)
  - R2L and U2R models were super fast (0.56s and 0.03s)

---

### **Key Decisions**  
1. **Algorithm Selection: One-Class SVM**:  
   - **Approach**: Used One-Class SVM because it lets us control the fraction of support vectors through the nu parameter.
   - **Trade-off**: Takes longer to train than Isolation Forest, especially for big classes like DOS.
   - **Impact**: Got high precision for most classes but terrible recall for normal traffic.

2. **Class-Specific Nu Parameters**:  
   - **Strategy**: Used lower nu (0.1) for normal traffic and higher nu (0.2) for attack classes.
   - **Result**: Created a boundary that's too tight around normal traffic, so almost none were correctly identified.
   - **Evidence**: Only 10 out of 9,711 normal samples were correctly classified.

3. **Sample Balancing Strategy**:  
   - **Decision**: Used different normal-to-attack ratios based on class size.
   - **Impact**: Worked okay for DOS and Probe, but didn't have enough data for R2L and U2R.

4. **Score-Based Classification Approach**:  
   - **Method**: Classified samples based on which model gave the lowest anomaly score.
   - **Problem**: This created a huge bias toward classifying things as DOS.
   - **Result**: Almost all samples (21,340 out of 22,544) ended up classified as either normal or DOS.

---

### **Conclusion**  
The One-Class SVM approach (ocsvm_multi_v1) performed worse than both Isolation Forest models, with only 36.6% accuracy compared to 39.9% for if_multi_v1 and 62.8% for if_multi_v2. The biggest problem was that it failed to identify normal traffic (only 0.1% detected correctly) while being extremely biased toward classifying everything as DOS attacks.

The OCSVM model showed mixed performance across classes:
- Extremely poor detection of normal traffic (0.1% vs 2.4% in if_multi_v1)
- Better DOS detection than both IF models (99.6% recall)
- Worse Probe and R2L detection than both IF models
- Better U2R recall than if_multi_v1 but worse than if_multi_v2

The main limitations were the extremely tight boundary around normal traffic and the computational demands of training the DOS model, which took nearly 10 minutes.

### **Future Work**
1. **Nu Parameter Tuning**:
   - Increase the nu parameter for normal traffic (from 0.1 to 0.3-0.4) to make the boundary less strict.
   - Test different nu values for attack classes to find the best balance between precision and recall.
   - Run a grid search to systematically find optimal nu parameters for each class.

2. **Kernel Selection and Optimization**:
   - Try linear or polynomial kernels instead of RBF to reduce training time and prevent overfitting.
   - Experiment with different gamma values for the RBF kernel.
   - Implement automated gamma parameter selection using cross-validation.
   - Test feature scaling techniques that might improve kernel performance.