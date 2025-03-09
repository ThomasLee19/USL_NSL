### **Logbook: Network Intrusion Detection Dataset Preprocessing for Multiclass Classification**  
**File Name**: `dataset_preprocess_multi`  

---

#### **Date**: February 11, 2025
**Objective**: Process the NSL-KDD dataset for multiclass network intrusion detection, converting raw data into a structured format suitable for training unsupervised anomaly detection models with five distinct categories: Normal Traffic and four attack types (DOS, Probe, R2L, and U2R).  

---

### **Work Summary**  
1. **Data Loading & Class Mapping Configuration**  
   - **Raw Data Source**: Loaded KDDTrain+ and KDDTest+ datasets with 42 features.
   - **Attack Categorization Scheme**:  
     - Class 0 (Normal Traffic): normal traffic only
     - Class 1 (DOS - Denial of Service): back, land, neptune, pod, smurf, teardrop, apache2, udpstorm, processtable, mailbomb
     - Class 2 (Probe - Surveillance/Scanning): ipsweep, nmap, portsweep, satan, mscan, saint
     - Class 3 (R2L - Remote to Local): ftp_write, guess_passwd, imap, multihop, phf, spy, warezclient, warezmaster, etc.
     - Class 4 (U2R - User to Root): buffer_overflow, loadmodule, perl, rootkit, sqlattack, xterm, ps
   - **Raw Data Statistics**:
     - Training set: 125,973 samples with 42 features
     - Major attack types: normal (53.5%), neptune (32.7%), satan (2.9%), ipsweep (2.9%)
     - Minor attack types: multihop (0.006%), phf (0.003%), perl (0.002%), spy (0.002%)

2. **Feature Preprocessing**  
   - **Feature Scaling**: Applied StandardScaler to all numeric features to normalize their distribution.
   - **Categorical Encoding**: Converted categorical variables (protocol_type, service, flag) to dummy variables using one-hot encoding.
   - **Scaling Outcome**: Transformed features to have approximately zero mean and unit variance, improving model performance.
   - **One-Hot Encoding Result**: Expanded the feature space from 42 to 123 dimensions to properly represent categorical variables.

3. **Multiclass Label Generation**  
   - **Label Mapping**: Created multiclass labels by mapping each attack type to one of the five categories.
   - **Class Distribution**:
     ```
     Class 0 (Normal Traffic): 67,343 samples (53.46%)
     Class 1 (DOS): 45,927 samples (36.46%)
     Class 2 (Probe): 11,656 samples (9.25%)
     Class 3 (R2L): 995 samples (0.79%)
     Class 4 (U2R): 52 samples (0.04%)
     ```
   - **Imbalance Analysis**: Severe class imbalance observed with normal:U2R ratio of 1,295:1 in training data.

4. **Feature Selection & Dimensionality Reduction**  
   - **Selection Method**: Applied VarianceThreshold with threshold=0.1 to remove low-variance features.
   - **Feature Reduction**: Reduced feature space from 123 to 43 dimensions.
   - **Selected Features**: Retained key features including traffic volume metrics, error rates, connection patterns, and protocol indicators.
   - **Final Dataset Format**: Both training and test datasets saved with 43 feature columns plus 1 multiclass label column.

---

### **Results**  
- **Final Dataset Shapes**: 
  | Dataset    | Samples | Features |
  |------------|---------|----------|
  | Training   | 125,973 | 44       |
  | Testing    | 22,544  | 44       |

- **Class Distribution in Test Dataset**:
  | Class | Description                  | Samples | Percentage |
  |-------|------------------------------|---------|------------|
  | 0     | Normal Traffic               | 9,711   | 43.08%     |
  | 1     | DOS (Denial of Service)      | 7,458   | 33.08%     |
  | 2     | Probe (Surveillance/Scanning)| 2,421   | 10.74%     |
  | 3     | R2L (Remote to Local)        | 2,887   | 12.81%     |
  | 4     | U2R (User to Root)           | 67      | 0.30%      |

- **Class Imbalance Analysis**:  
  - Training set imbalance ratio (max/min): 1,295.06
  - Test set imbalance ratio (max/min): 144.94
  - Class 3 (R2L) has 16× higher representation in test set (12.81%) compared to training set (0.79%)
  - Class 4 (U2R) has 7.5× higher representation in test set (0.30%) compared to training set (0.04%)

- **Key Feature Statistics by Class**:
  ```
  Feature: count
    Class 0 (Normal Traffic): mean=-0.5379, std=0.4718
    Class 1 (DOS): mean=0.8207, std=0.9121
    Class 2 (Probe): mean=-0.0616, std=1.3665
    Class 3 (R2L): mean=-0.7232, std=0.0042
    Class 4 (U2R): mean=-0.6838, std=0.2048

  Feature: diff_srv_rate
    Class 0 (Normal Traffic): mean=-0.1900, std=0.8076
    Class 1 (DOS): mean=0.0130, std=0.3551
    Class 2 (Probe): mean=1.0732, std=2.2694
    Class 3 (R2L): mean=-0.3125, std=0.4452
    Class 4 (U2R): mean=0.0033, std=1.2274
  ```

---

### **Key Decisions**  
1. **Multiclass Attack Categorization**:  
   - **Approach**: Grouped 38 different attack types into 4 distinct attack categories based on attack characteristics.
   - **Rationale**: Simplifies the problem while maintaining meaningful security categories recognized in cybersecurity literature.
   - **Impact**: Enables more effective model training by providing sufficient samples for each major attack category.
   - **Challenge**: Creates severely imbalanced classes, especially for U2R attacks (only 52 training samples).

2. **Feature Transformation Strategy**:  
   - **Decision**: Used StandardScaler for numeric features and one-hot encoding for categorical features.
   - **Reasoning**: Ensures all numeric features have comparable scales and properly represents categorical variables.
   - **Alternative Considered**: MinMaxScaler and ordinal encoding, but these would not preserve statistical properties or properly represent categorical relationships.
   - **Outcome**: Features with good statistical properties for both distance-based and tree-based algorithms.

3. **Feature Selection Approach**:  
   - **Method**: Applied VarianceThreshold with threshold=0.1, selecting only features with sufficient variance.
   - **Justification**: Low-variance features add little discriminative power and can cause numerical stability issues.
   - **Trade-off**: Reduced dimensionality improves model efficiency but potentially loses some rare-case indicators.
   - **Result**: Reduced feature space by 65% while retaining most informative features.

4. **Test Set Processing Strategy**:  
   - **Approach**: Applied the same transformations to test data using parameters fitted on training data.
   - **Handling Unknowns**: Implemented logic to automatically map new attack types to the most appropriate category.
   - **Importance**: Ensures consistent feature representation between training and test data, critical for valid evaluation.
   - **Verification**: Checked transformed test set for consistency in feature ranges and distributions.

---

### **Conclusion**  
The preprocessing pipeline successfully transformed the raw NSL-KDD dataset into a structured format suitable for training multiclass anomaly detection models. The final dataset contains 43 selected features plus multiclass labels for 125,973 training samples and 22,544 test samples.

The main challenges observed during preprocessing include:

1. **Severe Class Imbalance**: The rarest attack class (U2R) is represented by only 52 samples in the training set, which will likely require special handling during model training.

2. **Distribution Shift**: Significant differences in class distributions between training and test sets, particularly for R2L attacks (0.79% in training vs. 12.81% in testing), which may impact model generalization.

3. **Feature Engineering Requirements**: The transformed dataset provides a good starting point, but additional feature engineering might be necessary to improve detection of underrepresented attack types.

4. **Data Quality Considerations**: Some features show zero variance within certain classes (e.g., very low std=0.0042 for 'count' in R2L class), suggesting limited feature variability for these attacks.

These challenges will need to be addressed in subsequent modeling steps through techniques such as oversampling, specialized loss functions, or ensemble methods. The preprocessing results also indicate that different attack classes have distinct statistical signatures, suggesting that class-specific models might be effective for this detection task.