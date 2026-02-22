# Titanic Dataset – Basic Exploratory Data Analysis (EDA)

**Model:** `df` – 891 rows × 12 columns  
**Target:** `Survived` (0 = No, 1 = Yes)  

---

## 1️⃣ Dataset Overview  

| Feature | Type | Description | Missing Values |
|---|---|---|---|
| **PassengerId** | int64 | Unique identifier for each passenger | 0 |
| **Survived** | int64 | Survival outcome (target) | 0 |
| **Pclass** | int64 | Ticket class (1 = 1st, 2 = 2nd, 3 = 3rd) | 0 |
| **Name** | object | Full passenger name | 0 |
| **Sex** | object | Gender (male / female) | 0 |
| **Age** | float64 | Age in years (missing for 177 rows) | 177 (19.9 %) |
| **SibSp** | int64 | # of siblings / spouses aboard | 0 |
| **Parch** | int64 | # of parents / children aboard | 0 |
| **Ticket** | object | Ticket number (highly unique) | 0 |
| **Fare** | float64 | Ticket fare (missing for 0 rows) | 0 |
| **Cabin** | object | Cabin identifier (missing for 687 rows) | 687 (76.9 %) |
| **Embarked** | object | Port of embarkation (C = Cherbourg, Q = Queenstown, S = Southampton) | 2 (0.22 %) |

**Key Stats (from `describe`)**

| Feature | Mean | Std | Min | 25% | 50% | 75% | Max |
|---|---|---|---|---|---|---|---|
| PassengerId | 446.0 | 257.35 | 1 | 223.5 | 446 | 668.5 | 891 |
| Survived | 0.3838 | 0.4866 | 0 | 0 | 0 | 1 | 1 |
| Pclass | 2.3086 | 0.8361 | 1 | 2 | 3 | 3 | 3 |
| Age | 29.70 | 14.53 | 0.42 | 20.125 | 28.0 | 38.0 | 80.0 |
| SibSp | 0.523 | 1.103 | 0 | 0 | 0 | 1 | 8 |
| Parch | 0.3816 | 0.806 | 0 | 0 | 0 | 0 | 6 |
| Fare | 32.2042 | 49.693 | 0.0 | 7.9104 | 14.4542 | 31.0 | 512.3292 |

**Skewness (positive values indicate right‑skewed)**  

- Survived: **0.479** (moderate skew)  
- Age: **0.389** (right‑skewed)  
- SibSp: **3.70** (high skew)  
- Parch: **2.75** (high skew)  
- Fare: **4.79** (very right‑skewed)  

**Outliers (detected by IQR‑based rule)**  

| Feature | # Outliers |
|---|---|
| Age | 11 |
| SibSp | 46 |
| Parch | 213 |
| Fare | 116 |

> *Note:* The outlier counts are unusually high for SibSp/Parch/Fare because the raw values are not bounded; many extreme values are simply rare combinations of family size and fare.

---

## 2️⃣ Feature Types & Initial Observations  

### Numeric Features  

| Feature | Observations |
|---|---|
| **PassengerId** | Pure identifier – no predictive signal. |
| **Pclass** | Ordinal, strongly linked to survival (higher class → higher survival). |
| **Age** | Continuous, right‑skewed; many missing values; children (< 16) have higher survival rates. |
| **SibSp** | Counts siblings/spouses; median 0, but a few passengers traveled with large families. |
| **Parch** | Counts parents/children; similar distribution to SibSp. |
| **Fare** | Continuous, heavy right‑skew; correlated with Pclass and family size. |

### Categorical / Mixed Features  

| Feature | Observations |
|---|---|
| **Sex** | Binary; historically a strong predictor (female ≈ 74 % survival). |
| **Embarked** | 3 categories, low missingness; small effect but can capture port‑specific boarding patterns. |
| **Name** | Raw string – not directly useful, but can be parsed for titles (Mr, Mrs, Miss, Master). |
| **Ticket** | Highly unique identifiers; unlikely to carry predictive information. |
| **Cabin** | Mostly missing; when present, provides cabin location (deck) which is linked to class, but missingness makes it unreliable for modelling. |

---

## 3️⃣ Distribution & Relationship Insights  

### 3.1 Survival by **Pclass** (known from Titanic literature)

| Pclass | Survival Rate (approx.) |
|---|---|
| 1st | ~62 % |
| 2nd | ~47 % |
| 3rd | ~24 % |

*Implication*: **Pclass** is a very strong predictor and should be retained (preferably as an ordinal variable).

### 3.2 Survival by **Sex**

| Sex | Survival Rate |
|---|---|
| Female | ~74 % |
| Male | ~18 % |

*Implication*: **Sex** is the single most important categorical predictor.

### 3.3 Age & Survival  

- **Children (≤ 16)** show a markedly higher survival probability (≈ 50 % +).  
- Median age (28) suggests a broad adult population; age distribution is right‑skewed, with a long tail of older passengers.

### 3.4 Family Size (SibSp + Parch)

| Family Size (SibSp+Parch+1) | Survival Rate (approx.) |
|---|---|
| 1 (alone) | ~30 % |
| 2‑4 | ~40‑50 % |
| ≥5 | ~15 % |

*Implication*: Family size is informative; consider creating a derived feature `FamilySize = SibSp + Parch + 1`.  

### 3.5 Fare & Survival  

- Higher fare groups (e.g., > 50) are almost always in **1st class** and show higher survival.  
- Fare distribution is highly skewed; a log‑transform (`log1p(Fare)`) often stabilises it.

### 3.6 Embarked  

- Slight differences in survival across ports: Cherbourg (C) > Southampton (S) > Queenstown (Q).  
- With only 2 missing values, simple mode imputation (most common: **S**) is sufficient.

---

## 4️⃣ Feature Importance & Candidate Drops  

### Features **to keep** (high predictive value)

| Feature | Reason |
|---|---|
| **Survived** | Target variable |
| **Pclass** | Strong class effect |
| **Sex** | Strong gender effect |
| **Age** (imputed) | Children vs adults; continuous predictor |
| **FamilySize** (derived) | Captures both SibSp & Parch information |
| **Fare** (log‑transformed) | Proxy for socioeconomic status |
| **Embarked** | Minor but useful; low missingness |
| **Title** (extracted from `Name`) | May encode social status (e.g., “Mrs”, “Miss”, “Master”) |

### Features **to drop** (low/no predictive signal)

| Feature | Reason |
|---|---|
| **PassengerId** | Unique identifier, no relation to survival |
| **Ticket** | Highly unique, not correlated with outcome |
| **Cabin** | 76.9 % missing; remaining values are noisy and mostly redundant with Pclass |
| **Name** (raw) | Directly not useful; only useful after extracting titles (which we will handle separately) |
| **SibSp** & **Parch** (as separate columns) | Redundant when combined into `FamilySize`; keeping both may cause multicollinearity |

> **Drop‑list** (immediate): `PassengerId`, `Ticket`, `Cabin`, `Name`.  
> **Optional keep** (if you plan to engineer new features): `SibSp`, `Parch` (for title extraction or family‑size derivation).  

---

## 5️⃣ Recommendations for Pre‑processing  

1. **Missing‑Value Imputation**  
   - `Age`: median (or more sophisticated: using `Pclass`, `Sex`, `Embarked` as predictors).  
   - `Embarked`: mode (“S”).  
   - `Cabin`: drop (or create a binary flag “HasCabin” if you want to capture any cabin presence).  

2. **Feature Engineering**  
   - `FamilySize = SibSp + Parch + 1`  
   - `IsAlone = 1 if FamilySize == 1 else 0`  
   - `Title = Name.split(',')[1].split('.')[0].strip()` → one‑hot encode (Mr, Mrs, Miss, Master, etc.).  
   - `logFare = np.log1p(Fare)` to reduce skew.  

3. **Encoding**  
   - `Pclass` → keep as ordinal (1‑3) or one‑hot.  
   - `Sex` → binary (male/female).  
   - `Embarked` → one‑hot (C, Q, S).  
   - `Title` → one‑hot or treat as ordinal (e.g., “Master” highest priority).  

4. **Outlier Handling**  
   - For `Fare`, cap extreme values at the 99th percentile (e.g., `Fare = np.clip(Fare, 0, fare_99th)`).  
   - For `Age`, cap at realistic bounds (0‑80) or use robust imputation.  

5. **Scaling / Normalisation**  
   - `logFare` and `Age` can be standardised for linear models or tree‑based models (no scaling needed for trees).  

---

## 6️⃣ Quick Correlation Snapshot (after cleaning)

| Feature | Survived |
|---|---|
| **Pclass** | ~‑0.34 (negative) |
| **Sex_female** | ~+0.55 |
| **Age** | ~‑0.07 (weak) |
| **FamilySize** | ~‑0.02 (very weak) |
| **logFare** | ~+0.25 |
| **Embarked_Q** | ~‑0.01 |
| **Embarked_S** | ~+0.02 |
| **Embarked_C** | ~+0.03 |

> *These values are typical for the classic Titanic dataset; exact numbers will be recomputed after imputation and feature engineering.*

---

## 7️⃣ Summary  

- **Core predictive variables**: `Pclass`, `Sex`, `Age` (imputed), `logFare`, `FamilySize` (derived).  
- **Features with negligible predictive power**: `PassengerId`, `Ticket`, `Cabin`, raw `Name`.  
- **Missingness**: Age (19.9 %) and Cabin (76.9 %) are the biggest gaps; impute Age, drop Cabin.  
- **Skew**: Fare and Parch are heavily right‑skewed; consider log‑transform or binning.  
- **Potential multicollinearity**: `SibSp` & `Parch` can be merged into `FamilySize`.  
- **Next steps**: Build a clean feature set, engineer `Title`, apply one‑hot/ordinal encoding, split data, and test baseline models (Logistic Regression, Random Forest, XGBoost).  

--- 

*Prepared as a senior‑level data‑science briefing for the Titanic survival prediction task.*