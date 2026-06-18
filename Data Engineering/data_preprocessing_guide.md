# 🧹 Data Preprocessing: A Complete Beginner's Guide

> *"Garbage in, garbage out."* — Every data scientist ever.  
> Preprocessing is what stands between garbage and great models.

---

## 📋 Table of Contents

1. [Problems Faced Before Preprocessing](#problems-faced-before-preprocessing)
2. [Preprocessing Techniques](#preprocessing-techniques)
   - [Handling Missing Values](#1-handling-missing-values)
   - [Handling Duplicates](#2-handling-duplicates)
   - [Handling Outliers](#3-handling-outliers)
   - [Encoding Categorical Variables](#4-encoding-categorical-variables)
   - [Feature Scaling / Normalization](#5-feature-scaling--normalization)
   - [Handling Imbalanced Data](#6-handling-imbalanced-data)
   - [Feature Engineering](#7-feature-engineering)
   - [Handling Date & Time Features](#8-handling-date--time-features)
   - [Text Preprocessing](#9-text-preprocessing)
   - [Handling Skewed Data](#10-handling-skewed-data)
3. [Generic PreprocessorManager Class](#generic-preprocessormanager-class)

---

## ⚠️ Problems Faced Before Preprocessing

Real-world data is messy. Here are the most common problems and their high-level solutions:

| # | Problem | Description | Solution |
|---|---------|-------------|----------|
| 1 | **Missing Values** | Cells with `NaN`, `null`, or blank entries | Imputation or removal |
| 2 | **Duplicate Records** | Same row appearing multiple times | Remove duplicates |
| 3 | **Outliers** | Extreme values far from normal range | Cap, transform, or remove |
| 4 | **Categorical Data** | Text labels like `"Male"`, `"Red"` | Encode to numbers |
| 5 | **Different Scales** | Age in [0–100], Salary in [10000–100000] | Normalize / Standardize |
| 6 | **Imbalanced Classes** | 95% class A, 5% class B | Oversample, undersample, or use weights |
| 7 | **Irrelevant Features** | Columns that add no predictive value | Drop or transform |
| 8 | **Skewed Distributions** | Most values piled on one side | Log / Box-Cox transform |
| 9 | **Raw Date/Time** | `"2023-04-15"` as a single string | Extract day, month, year, weekday, etc. |
| 10 | **Raw Text** | Unstructured natural language | Tokenize, clean, vectorize |
| 11 | **Mixed Data Types** | A numeric column stored as string | Type casting |
| 12 | **Inconsistent Formatting** | `"new york"`, `"New York"`, `"NY"` | Standardize values |

---

## 🛠️ Preprocessing Techniques

---

### 1. Handling Missing Values

#### 📌 What it does
Identifies columns/rows where data is absent (`NaN`, `None`, empty string) and fills or removes them.

#### 🤔 Why it is done
Most ML algorithms cannot handle `NaN` values — they will either crash or produce wrong results. Missing data must be resolved before training.

#### 📂 On what types of data
- **Numerical columns**: Fill with mean, median, or a constant.
- **Categorical columns**: Fill with mode (most frequent value) or a constant like `"Unknown"`.
- **Any column**: Drop rows/columns if too much data is missing (e.g., >50%).

#### 💻 Implementation

```python
import pandas as pd
import numpy as np
from sklearn.impute import SimpleImputer

df = pd.DataFrame({
    'Age':    [25, np.nan, 35, 40, np.nan],
    'Salary': [50000, 60000, np.nan, 80000, 55000],
    'Gender': ['Male', 'Female', np.nan, 'Female', 'Male']
})

# --- Check missing values ---
print(df.isnull().sum())
# Age       2
# Salary    1
# Gender    1

# --- Strategy 1: Drop rows with ANY missing value ---
df_dropped = df.dropna()

# --- Strategy 2: Drop columns with > 50% missing data ---
threshold = len(df) * 0.5
df_col_dropped = df.dropna(thresh=threshold, axis=1)

# --- Strategy 3: Fill numerical with MEAN using sklearn ---
num_imputer = SimpleImputer(strategy='mean')
df[['Age', 'Salary']] = num_imputer.fit_transform(df[['Age', 'Salary']])
# 'mean' -> fills NaN with column average
# 'median' -> robust to outliers, better choice often
# 'most_frequent' -> works for both num and categorical

# --- Strategy 4: Fill categorical with MODE ---
cat_imputer = SimpleImputer(strategy='most_frequent')
df[['Gender']] = cat_imputer.fit_transform(df[['Gender']])

# --- Strategy 5: Fill with a constant ---
df['Gender'].fillna('Unknown', inplace=True)

print(df)
```

**Explanation:**
- `isnull().sum()` → counts missing values per column.
- `dropna()` → removes rows; use cautiously if data is scarce.
- `SimpleImputer(strategy='mean')` → replaces NaN with mean of that column.
- `fit_transform()` → learns the mean from data, then applies it.
- Use **median** over mean when outliers exist — mean gets pulled by extremes.

---

### 2. Handling Duplicates

#### 📌 What it does
Detects and removes rows that are exact (or near-exact) copies of other rows in the dataset.

#### 🤔 Why it is done
Duplicate rows bias the model — it "sees" certain data points multiple times, leading to overfitting or unfair weight on those samples.

#### 📂 On what types of data
Applies to **all datasets** regardless of column types. Common in data collected from multiple sources, web scraping, or manual entry.

#### 💻 Implementation

```python
import pandas as pd

df = pd.DataFrame({
    'Name':   ['Alice', 'Bob', 'Alice', 'Charlie', 'Bob'],
    'Age':    [25, 30, 25, 35, 30],
    'Salary': [50000, 60000, 50000, 70000, 60000]
})

# --- Check for duplicates ---
print(df.duplicated().sum())  # Output: 2 (rows 2 and 4 are duplicates)

# --- View duplicate rows ---
print(df[df.duplicated(keep=False)])
# keep=False -> marks ALL occurrences of duplicates as True

# --- Remove duplicates (keep first occurrence) ---
df_clean = df.drop_duplicates(keep='first')
# keep='first'  -> retains first occurrence
# keep='last'   -> retains last occurrence
# keep=False    -> removes ALL duplicate occurrences

# --- Check duplicates on specific columns only ---
df_clean2 = df.drop_duplicates(subset=['Name', 'Age'], keep='first')

print(df_clean)
```

**Explanation:**
- `duplicated()` → returns a boolean Series, `True` where row is a duplicate.
- `drop_duplicates()` → removes duplicate rows in-place or returns new df.
- `subset` → consider only specific columns to define "duplicate" (useful when you only want unique `Name+Age` combos regardless of salary).

---

### 3. Handling Outliers

#### 📌 What it does
Detects data points that are unusually far from the rest of the distribution and either removes, caps, or transforms them.

#### 🤔 Why it is done
Outliers distort statistics (mean, variance) and confuse ML models — especially linear models and KNN which are distance-sensitive.

#### 📂 On what types of data
**Numerical columns only.** Outliers don't apply to categorical data.

#### 💻 Implementation

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'Age':    [22, 25, 27, 30, 28, 150, 24, 29],  # 150 is an outlier
    'Salary': [30000, 45000, 50000, 55000, 48000, 52000, 1000000, 47000]  # 1M is outlier
})

# ============================
# METHOD 1: IQR (Interquartile Range)
# ============================
def remove_outliers_iqr(df, column):
    Q1 = df[column].quantile(0.25)        # 25th percentile
    Q3 = df[column].quantile(0.75)        # 75th percentile
    IQR = Q3 - Q1                          # Interquartile range
    lower = Q1 - 1.5 * IQR               # Standard fence: 1.5 * IQR below Q1
    upper = Q3 + 1.5 * IQR               # Standard fence: 1.5 * IQR above Q3
    return df[(df[column] >= lower) & (df[column] <= upper)]

df_clean_iqr = remove_outliers_iqr(df, 'Age')
print("After IQR:", df_clean_iqr)

# ============================
# METHOD 2: Z-Score
# ============================
from scipy import stats

z_scores = np.abs(stats.zscore(df['Salary']))
# z > 3 means value is > 3 standard deviations from mean → outlier
df_clean_z = df[z_scores < 3]
print("After Z-Score:", df_clean_z)

# ============================
# METHOD 3: Capping (Winsorization) — keeps row but clips value
# ============================
lower = df['Age'].quantile(0.05)    # 5th percentile as lower bound
upper = df['Age'].quantile(0.95)    # 95th percentile as upper bound
df['Age_capped'] = df['Age'].clip(lower=lower, upper=upper)
# Values below lower → set to lower; above upper → set to upper
print(df[['Age', 'Age_capped']])
```

**Explanation:**
- **IQR method**: Uses quartiles — robust to skewed data. Anything outside `[Q1 - 1.5*IQR, Q3 + 1.5*IQR]` is an outlier.
- **Z-Score method**: Assumes normal distribution. Values with `|z| > 3` are flagged. Best for roughly symmetric data.
- **Capping (Winsorization)**: Doesn't remove rows — instead clips extreme values to a boundary. Safer when you can't afford to lose data.

---

### 4. Encoding Categorical Variables

#### 📌 What it does
Converts text/label categories into numerical format so ML algorithms can process them.

#### 🤔 Why it is done
ML models are mathematical — they work with numbers. Strings like `"Red"`, `"Blue"` must be converted to numeric representations.

#### 📂 On what types of data
**Categorical columns** — both nominal (no order: `Red, Blue, Green`) and ordinal (ordered: `Low, Medium, High`).

#### 💻 Implementation

```python
import pandas as pd
from sklearn.preprocessing import LabelEncoder, OrdinalEncoder
from sklearn.preprocessing import OneHotEncoder

df = pd.DataFrame({
    'Color':       ['Red', 'Blue', 'Green', 'Blue', 'Red'],
    'Size':        ['Small', 'Large', 'Medium', 'Small', 'Large'],
    'Brand':       ['Nike', 'Adidas', 'Nike', 'Puma', 'Adidas']
})

# ============================
# METHOD 1: Label Encoding
# (Use ONLY for ordinal data or binary categories)
# ============================
le = LabelEncoder()
df['Size_encoded'] = le.fit_transform(df['Size'])
# Small→0, Large→1, Medium→2  (arbitrary — model may think Large > Small > Medium which is wrong)
# ONLY use this for truly ordinal data like Low/Medium/High

# ============================
# METHOD 2: Ordinal Encoding (for ordered categories)
# ============================
oe = OrdinalEncoder(categories=[['Small', 'Medium', 'Large']])
df['Size_ordinal'] = oe.fit_transform(df[['Size']])
# Small→0, Medium→1, Large→2  (correctly preserves order!)

# ============================
# METHOD 3: One-Hot Encoding (for nominal categories — NO order)
# ============================
df_ohe = pd.get_dummies(df, columns=['Color', 'Brand'], drop_first=True)
# drop_first=True -> avoids dummy variable trap (multicollinearity)
# Creates: Color_Blue, Color_Green (Red is reference), Brand_Nike, Brand_Puma (Adidas is reference)
print(df_ohe)

# Using sklearn's OneHotEncoder:
ohe = OneHotEncoder(drop='first', sparse_output=False)
encoded = ohe.fit_transform(df[['Color']])
print(ohe.get_feature_names_out())  # ['Color_Blue', 'Color_Green']

# ============================
# METHOD 4: Frequency Encoding (for high-cardinality columns)
# (replaces each category with its frequency ratio in data)
# ============================
freq_map = df['Brand'].value_counts(normalize=True)   # proportion of each brand
df['Brand_freq'] = df['Brand'].map(freq_map)
# Nike=0.4, Adidas=0.4, Puma=0.2 (based on occurrences)
print(df[['Brand', 'Brand_freq']])
```

**Explanation:**
- **Label Encoding**: Simple integer mapping. ⚠️ Dangerous for nominal data — implies an order that doesn't exist.
- **Ordinal Encoding**: Correctly assigns integers in your specified order. Best for `Low/Medium/High`.
- **One-Hot Encoding**: Creates one binary column per category. Best for nominal data. `drop_first=True` removes one column to avoid redundancy.
- **Frequency Encoding**: Encodes by how often each value appears. Great when there are many unique categories (high cardinality).

---

### 5. Feature Scaling / Normalization

#### 📌 What it does
Brings all numerical features to a comparable scale so no single feature dominates due to its unit of measurement.

#### 🤔 Why it is done
Algorithms like KNN, SVM, Logistic Regression, and Neural Networks are sensitive to feature scale. A `Salary` column in thousands will dominate `Age` in tens unless both are scaled.

#### 📂 On what types of data
**Numerical columns only.** Tree-based models (Random Forest, XGBoost) do NOT require scaling.

#### 💻 Implementation

```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import MinMaxScaler, StandardScaler, RobustScaler

df = pd.DataFrame({
    'Age':    [22, 35, 58, 45, 30],
    'Salary': [25000, 50000, 120000, 75000, 40000]
})

# ============================
# METHOD 1: Min-Max Scaling (Normalization)
# Scales data to range [0, 1]
# Formula: (x - min) / (max - min)
# ============================
scaler_minmax = MinMaxScaler()
df_minmax = pd.DataFrame(
    scaler_minmax.fit_transform(df),
    columns=df.columns
)
print("Min-Max:\n", df_minmax)
# Age values become: 0.0, 0.361, 1.0, 0.638, 0.222

# ============================
# METHOD 2: Standard Scaling (Standardization)
# Scales data to mean=0 and std=1
# Formula: (x - mean) / std
# Best when data is roughly Gaussian (bell-curve shaped)
# ============================
scaler_std = StandardScaler()
df_std = pd.DataFrame(
    scaler_std.fit_transform(df),
    columns=df.columns
)
print("Standard Scaled:\n", df_std)

# ============================
# METHOD 3: Robust Scaling
# Uses median and IQR instead of mean and std
# Best when data HAS outliers
# ============================
scaler_robust = RobustScaler()
df_robust = pd.DataFrame(
    scaler_robust.fit_transform(df),
    columns=df.columns
)
print("Robust Scaled:\n", df_robust)
```

**Explanation:**
- **Min-Max**: Compresses all values into `[0, 1]`. Sensitive to outliers because `min` and `max` are used.
- **Standard Scaler**: Centers data at 0 with unit variance. Best for normally distributed features.
- **Robust Scaler**: Uses `median` and `IQR` — much less affected by outliers. Use when outlier removal isn't possible.

**Quick Guide:**
| Scenario | Use |
|----------|-----|
| No outliers, neural networks / KNN | MinMax |
| Data is Gaussian, SVM / Logistic Regression | Standard |
| Outliers present | Robust |
| Tree-based models | No scaling needed |

---

### 6. Handling Imbalanced Data

#### 📌 What it does
Balances the class distribution when one class is far more frequent than another in a classification problem.

#### 🤔 Why it is done
A model trained on 99% Class A and 1% Class B will learn to always predict A and still achieve 99% accuracy — but it's useless. Balancing ensures the model learns from both classes.

#### 📂 On what types of data
**Target/label column in classification problems.** (Not applied to regression.)

#### 💻 Implementation

```python
import pandas as pd
from sklearn.utils import resample
from imblearn.over_sampling import SMOTE

# Simulated imbalanced dataset
df = pd.DataFrame({
    'Feature1': [1,2,3,4,5,6,7,8,9,10,11,12],
    'Feature2': [10,20,30,40,50,60,70,80,90,100,110,120],
    'Label':    [0, 0, 0, 0, 0, 0, 0, 0, 0,  1,   1,  1]  # 9 zeros, 3 ones
})

print(df['Label'].value_counts())
# 0 → 9 samples  (majority)
# 1 → 3 samples  (minority)

# ============================
# METHOD 1: Oversampling Minority Class (Manual)
# Duplicates existing minority rows randomly
# ============================
majority = df[df['Label'] == 0]
minority = df[df['Label'] == 1]

minority_upsampled = resample(minority,
    replace=True,           # Sample with replacement (allows duplicates)
    n_samples=len(majority), # Match majority class count
    random_state=42
)
df_balanced = pd.concat([majority, minority_upsampled])
print(df_balanced['Label'].value_counts())  # Now 9-9

# ============================
# METHOD 2: Undersampling Majority Class
# Removes rows from majority class randomly
# ============================
majority_downsampled = resample(majority,
    replace=False,
    n_samples=len(minority),  # Match minority count
    random_state=42
)
df_balanced2 = pd.concat([majority_downsampled, minority])
print(df_balanced2['Label'].value_counts())  # Now 3-3

# ============================
# METHOD 3: SMOTE (Synthetic Minority Over-sampling Technique)
# Generates NEW synthetic minority samples using KNN interpolation
# Best method — doesn't just duplicate, creates realistic new data
# ============================
X = df[['Feature1', 'Feature2']]
y = df['Label']

smote = SMOTE(random_state=42)
X_resampled, y_resampled = smote.fit_resample(X, y)
print(pd.Series(y_resampled).value_counts())  # Balanced!

# ============================
# METHOD 4: Class Weights in Model (No data modification)
# ============================
from sklearn.linear_model import LogisticRegression
model = LogisticRegression(class_weight='balanced')
# 'balanced' automatically adjusts weights inversely proportional to class frequency
```

**Explanation:**
- **Oversampling**: Duplicates minority rows — fast but may cause overfitting on minority class.
- **Undersampling**: Reduces majority rows — can lose important data.
- **SMOTE**: Creates synthetic samples by interpolating between existing minority points using K-Nearest Neighbors. Best of both worlds.
- **class_weight**: Tells the model to penalize misclassification of minority class more — no data change needed.

---

### 7. Feature Engineering

#### 📌 What it does
Creates new, more informative features from existing ones to improve model performance.

#### 🤔 Why it is done
Raw data rarely has the exact representation a model needs. A column `Height_cm` and `Weight_kg` are less useful than `BMI = weight / height²` which captures fitness better.

#### 📂 On what types of data
**Numerical, date/time, or categorical data.** Totally context-dependent.

#### 💻 Implementation

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'Height_cm':     [170, 165, 180, 155, 175],
    'Weight_kg':     [70, 55, 90, 60, 80],
    'Salary':        [50000, 60000, 80000, 45000, 75000],
    'Years_Exp':     [3, 5, 10, 2, 8],
    'Name':          ['Alice Bob', 'Charlie', 'Diana Prince', 'Eve', 'Frank Castle'],
    'Purchase_Date': ['2023-01-15', '2023-06-22', '2022-12-01', '2023-03-08', '2023-07-30']
})

# --- Interaction Feature ---
df['BMI'] = df['Weight_kg'] / (df['Height_cm'] / 100) ** 2
# BMI = weight(kg) / height(m)^2 → more meaningful health indicator

# --- Ratio Feature ---
df['Salary_Per_Year_Exp'] = df['Salary'] / df['Years_Exp']
# Earnings efficiency — how much per year of experience

# --- Polynomial Feature ---
df['Years_Exp_Squared'] = df['Years_Exp'] ** 2
# Captures diminishing/accelerating returns from experience

# --- Binning (Discretization) ---
df['BMI_Category'] = pd.cut(df['BMI'],
    bins=[0, 18.5, 24.9, 29.9, 100],
    labels=['Underweight', 'Normal', 'Overweight', 'Obese']
)
# Converts continuous BMI into clinically meaningful categories

# --- Text-derived Feature ---
df['Name_Word_Count'] = df['Name'].apply(lambda x: len(x.split()))
# Number of words in name (useful proxy for data quality)

print(df[['BMI', 'BMI_Category', 'Salary_Per_Year_Exp']])
```

**Explanation:**
- **Interaction features**: Combine two columns to create a new signal (`BMI` from height + weight).
- **Ratio features**: Normalize a value against another (`Salary per year of experience`).
- **Polynomial features**: Adds non-linear relationships without switching to a non-linear model.
- **Binning**: Groups continuous values into categorical bands — helps when the relationship is step-wise, not linear.

---

### 8. Handling Date & Time Features

#### 📌 What it does
Extracts meaningful components from raw date/time strings into separate numerical or categorical columns.

#### 🤔 Why it is done
Raw strings like `"2023-06-15"` are meaningless to models. Breaking them into `year=2023, month=6, day=15, weekday=Thursday` gives the model real temporal signals (e.g., sales are higher on weekends).

#### 📂 On what types of data
**Date, DateTime, or timestamp columns.**

#### 💻 Implementation

```python
import pandas as pd

df = pd.DataFrame({
    'Order_Date': ['2023-01-15 08:30:00', '2022-12-24 22:15:00',
                   '2023-06-01 12:00:00', '2023-03-08 09:45:00']
})

# --- Step 1: Parse string to datetime ---
df['Order_Date'] = pd.to_datetime(df['Order_Date'])

# --- Step 2: Extract components ---
df['Year']        = df['Order_Date'].dt.year
df['Month']       = df['Order_Date'].dt.month
df['Day']         = df['Order_Date'].dt.day
df['Hour']        = df['Order_Date'].dt.hour
df['Day_of_Week'] = df['Order_Date'].dt.dayofweek   # 0=Monday, 6=Sunday
df['Day_Name']    = df['Order_Date'].dt.day_name()   # 'Sunday', 'Monday', ...
df['Is_Weekend']  = df['Day_of_Week'].isin([5, 6]).astype(int)  # 1 if Sat/Sun
df['Quarter']     = df['Order_Date'].dt.quarter      # 1, 2, 3, or 4
df['Week_of_Year']= df['Order_Date'].dt.isocalendar().week.astype(int)

# --- Step 3: Time difference feature ---
reference_date = pd.Timestamp('2023-07-01')
df['Days_Since_Order'] = (reference_date - df['Order_Date']).dt.days

# --- Step 4: Cyclic encoding for cyclical features (e.g., month, hour) ---
# Month 12 and Month 1 are CLOSE, but numerically 12 and 1 are far apart.
# Cyclic encoding fixes this using sine and cosine:
import numpy as np
df['Month_Sin'] = np.sin(2 * np.pi * df['Month'] / 12)
df['Month_Cos'] = np.cos(2 * np.pi * df['Month'] / 12)

print(df[['Order_Date', 'Year', 'Month', 'Day', 'Is_Weekend', 'Days_Since_Order']])
```

**Explanation:**
- `pd.to_datetime()` → converts raw string into a proper datetime object.
- `.dt` accessor → unlocks all date/time extraction methods.
- `Is_Weekend` → binary flag from day of week; useful for e-commerce, traffic models.
- **Cyclic encoding** → sine/cosine encoding prevents the model from treating Dec (12) as far from Jan (1). Without this, months look linear when they're actually circular.

---

### 9. Text Preprocessing

#### 📌 What it does
Cleans and transforms raw natural language text into a numeric format that ML models can consume.

#### 🤔 Why it is done
Raw text contains noise — HTML tags, punctuation, casing differences, stopwords — that add no value. The model cannot understand the string `"Hello World!"`; it needs numbers.

#### 📂 On what types of data
**Free-text / NLP columns** — reviews, descriptions, comments, tweets, emails.

#### 💻 Implementation

```python
import pandas as pd
import re
import nltk
from nltk.corpus import stopwords
from nltk.stem import WordNetLemmatizer
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer

nltk.download('stopwords')
nltk.download('wordnet')
nltk.download('punkt')

df = pd.DataFrame({
    'Review': [
        "This product is AMAZING!! 😍 Best buy ever!!! <br>",
        "Absolutely terrible... won't buy again. http://spam.com",
        "It was okay, nothing special. The delivery was running quickly.",
        "Great product!!! highly recommend to everyone."
    ]
})

lemmatizer = WordNetLemmatizer()
stop_words = set(stopwords.words('english'))

def preprocess_text(text):
    # Step 1: Lowercase
    text = text.lower()

    # Step 2: Remove HTML tags
    text = re.sub(r'<.*?>', '', text)

    # Step 3: Remove URLs
    text = re.sub(r'http\S+|www\S+', '', text)

    # Step 4: Remove emojis and special characters (keep only letters and spaces)
    text = re.sub(r'[^a-z\s]', '', text)

    # Step 5: Remove extra whitespace
    text = re.sub(r'\s+', ' ', text).strip()

    # Step 6: Tokenize (split into words)
    tokens = text.split()

    # Step 7: Remove stopwords ('is', 'the', 'was', etc. — low information words)
    tokens = [word for word in tokens if word not in stop_words]

    # Step 8: Lemmatize (reduce to base form: 'running' → 'run', 'better' → 'good')
    tokens = [lemmatizer.lemmatize(word) for word in tokens]

    return ' '.join(tokens)

df['Cleaned_Review'] = df['Review'].apply(preprocess_text)
print(df[['Review', 'Cleaned_Review']])

# --- Vectorization: Convert text to numbers ---

# TF-IDF: Weighs words by how important they are (common words get lower weight)
tfidf = TfidfVectorizer(max_features=10)
X_tfidf = tfidf.fit_transform(df['Cleaned_Review'])
print("TF-IDF Feature Names:", tfidf.get_feature_names_out())
print("TF-IDF Matrix:\n", X_tfidf.toarray())

# Count Vectorizer: Simple word frequency count
cv = CountVectorizer(max_features=10)
X_cv = cv.fit_transform(df['Cleaned_Review'])
```

**Explanation:**
- **Lowercase**: Ensures `"Amazing"` and `"amazing"` are treated as the same word.
- **Remove HTML/URLs**: Common in web-scraped data — they're noise.
- **Regex cleaning**: Removes punctuation, emojis, and special characters.
- **Stopword removal**: Words like `"the"`, `"is"`, `"was"` appear everywhere but carry no useful meaning.
- **Lemmatization**: Reduces words to root form — `"running"` → `"run"`. Better than stemming (which can mangle words like `"studies"` → `"studi"`).
- **TF-IDF**: Creates a numeric matrix where each column is a word and each value represents how important that word is in that document relative to all documents.

---

### 10. Handling Skewed Data

#### 📌 What it does
Applies mathematical transformations to make highly skewed distributions more symmetric (closer to a normal/Gaussian distribution).

#### 🤔 Why it is done
Many algorithms assume normally distributed features. Skewed data inflates the effect of extreme values. Linear regression residuals should be symmetric — skewness violates this.

#### 📂 On what types of data
**Positive numerical columns** with skewness. Check with `df['col'].skew()` — skewness > 1 or < -1 is significant.

#### 💻 Implementation

```python
import pandas as pd
import numpy as np
from scipy import stats
from sklearn.preprocessing import PowerTransformer

df = pd.DataFrame({
    'Income':   [15000, 20000, 22000, 18000, 500000, 25000, 19000, 1000000, 21000, 16000]
})

print("Original Skewness:", df['Income'].skew())
# High positive skew — long right tail due to 500K and 1M outliers

# ============================
# METHOD 1: Log Transformation
# Best for right-skewed data (positive skew)
# Requires all values > 0; use log1p for zero-containing data
# ============================
df['Income_Log'] = np.log1p(df['Income'])    # log(1 + x) to handle zeros safely
print("Log Skewness:", df['Income_Log'].skew())

# ============================
# METHOD 2: Square Root Transformation
# Milder than log; good for moderate skew
# ============================
df['Income_Sqrt'] = np.sqrt(df['Income'])

# ============================
# METHOD 3: Box-Cox Transformation
# Automatically finds the best power transformation
# Requires all values > 0 (strictly positive)
# ============================
df['Income_BoxCox'], _ = stats.boxcox(df['Income'])
print("Box-Cox Skewness:", df['Income_BoxCox'].skew())

# ============================
# METHOD 4: Yeo-Johnson Transformation
# Like Box-Cox but works with negative values and zeros too
# ============================
pt = PowerTransformer(method='yeo-johnson')
df['Income_YeoJohnson'] = pt.fit_transform(df[['Income']])

print(df[['Income', 'Income_Log', 'Income_BoxCox', 'Income_YeoJohnson']].head())
```

**Explanation:**
- **Log transform**: Compresses the long right tail drastically. Most common choice for income, price, population data.
- `log1p(x)` = `log(1 + x)` — handles the case where some values are 0 (since `log(0)` is undefined).
- **Square root**: A gentler compression. Use when log feels too aggressive.
- **Box-Cox**: Tries multiple power values and picks the one that best achieves normality. Automatic and data-driven.
- **Yeo-Johnson**: The most flexible — works even with zero and negative values. Usually the safest default.

---

---

## 🏗️ Generic PreprocessorManager Class

This class takes raw data in, applies the pipeline of preprocessing steps, and returns clean, model-ready data.

```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler, LabelEncoder, OrdinalEncoder
from sklearn.impute import SimpleImputer
from scipy import stats
import re
import warnings
warnings.filterwarnings('ignore')


class PreprocessorManager:
    """
    A generic, end-to-end data preprocessing pipeline.

    Usage:
        pm = PreprocessorManager(df)
        pm.handle_missing_values()
        pm.remove_duplicates()
        pm.handle_outliers(columns=['Age', 'Salary'])
        pm.encode_categoricals()
        pm.scale_features()
        clean_df = pm.get_processed_data()

    Or use the one-shot method:
        clean_df = pm.run_full_pipeline(
            outlier_cols=['Age', 'Salary'],
            ordinal_cols={'Size': ['Small', 'Medium', 'Large']}
        )
    """

    def __init__(self, df: pd.DataFrame):
        """
        Initialize with a raw DataFrame.
        A copy is made to avoid modifying the original data.
        """
        self.df = df.copy()
        self.original_shape = df.shape
        self.label_encoders = {}
        self.scalers = {}
        self.log = []   # Track what was done


    # ------------------------------------------------------------------ #
    #  STEP 1: TYPE CASTING                                               #
    # ------------------------------------------------------------------ #
    def fix_dtypes(self, num_cols: list = None, cat_cols: list = None):
        """
        Casts columns to the correct data types.
        Auto-detects if lists not provided.

        Parameters:
            num_cols: list of columns to force-convert to numeric
            cat_cols: list of columns to force-convert to string/category
        """
        if num_cols:
            for col in num_cols:
                self.df[col] = pd.to_numeric(self.df[col], errors='coerce')
                # errors='coerce' → non-numeric values become NaN (handled in next step)
        if cat_cols:
            for col in cat_cols:
                self.df[col] = self.df[col].astype(str).str.strip().str.lower()
                # Standardize: strip spaces, lowercase for consistent matching

        self.log.append("fix_dtypes: Type conversions applied.")
        return self


    # ------------------------------------------------------------------ #
    #  STEP 2: HANDLE MISSING VALUES                                      #
    # ------------------------------------------------------------------ #
    def handle_missing_values(self, num_strategy: str = 'median',
                               cat_strategy: str = 'most_frequent',
                               drop_thresh: float = 0.6):
        """
        Handles missing values across the entire DataFrame.

        Parameters:
            num_strategy:  'mean', 'median', or 'constant' for numerical columns
            cat_strategy:  'most_frequent' or 'constant' for categorical columns
            drop_thresh:   Drop columns where fraction of missing > this threshold (0.6 = 60%)
        """
        # Drop columns with too many missing values
        missing_frac = self.df.isnull().mean()
        cols_to_drop = missing_frac[missing_frac > drop_thresh].index.tolist()
        if cols_to_drop:
            self.df.drop(columns=cols_to_drop, inplace=True)
            self.log.append(f"handle_missing_values: Dropped {cols_to_drop} (>{drop_thresh*100}% missing)")

        # Identify numerical and categorical columns
        num_cols = self.df.select_dtypes(include=[np.number]).columns.tolist()
        cat_cols = self.df.select_dtypes(include=['object', 'category']).columns.tolist()

        # Fill numerical missing values
        if num_cols:
            num_imputer = SimpleImputer(strategy=num_strategy)
            self.df[num_cols] = num_imputer.fit_transform(self.df[num_cols])

        # Fill categorical missing values
        if cat_cols:
            cat_imputer = SimpleImputer(strategy=cat_strategy)
            self.df[cat_cols] = cat_imputer.fit_transform(self.df[cat_cols])

        self.log.append(f"handle_missing_values: Num→{num_strategy}, Cat→{cat_strategy}")
        return self


    # ------------------------------------------------------------------ #
    #  STEP 3: REMOVE DUPLICATES                                          #
    # ------------------------------------------------------------------ #
    def remove_duplicates(self, subset: list = None, keep: str = 'first'):
        """
        Removes duplicate rows from the DataFrame.

        Parameters:
            subset: List of columns to consider for duplicate checking (None = all columns)
            keep:   'first', 'last', or False (remove all occurrences)
        """
        before = len(self.df)
        self.df.drop_duplicates(subset=subset, keep=keep, inplace=True)
        self.df.reset_index(drop=True, inplace=True)
        after = len(self.df)
        self.log.append(f"remove_duplicates: Removed {before - after} duplicate rows.")
        return self


    # ------------------------------------------------------------------ #
    #  STEP 4: HANDLE OUTLIERS (IQR Capping)                             #
    # ------------------------------------------------------------------ #
    def handle_outliers(self, columns: list = None, method: str = 'cap',
                        iqr_factor: float = 1.5):
        """
        Detects and handles outliers in specified numerical columns.

        Parameters:
            columns:    List of numerical columns to check. None = all numerical columns.
            method:     'cap' (winsorize) or 'remove' (drop rows with outliers)
            iqr_factor: Multiplier for IQR fence (default 1.5; use 3.0 for less aggressive)
        """
        if columns is None:
            columns = self.df.select_dtypes(include=[np.number]).columns.tolist()

        for col in columns:
            Q1 = self.df[col].quantile(0.25)
            Q3 = self.df[col].quantile(0.75)
            IQR = Q3 - Q1
            lower = Q1 - iqr_factor * IQR
            upper = Q3 + iqr_factor * IQR

            if method == 'cap':
                # Winsorize: clip values to boundary instead of removing rows
                self.df[col] = self.df[col].clip(lower=lower, upper=upper)
            elif method == 'remove':
                # Remove rows where any specified column is an outlier
                mask = (self.df[col] >= lower) & (self.df[col] <= upper)
                self.df = self.df[mask].reset_index(drop=True)

        self.log.append(f"handle_outliers: method={method}, columns={columns}")
        return self


    # ------------------------------------------------------------------ #
    #  STEP 5: ENCODE CATEGORICAL VARIABLES                               #
    # ------------------------------------------------------------------ #
    def encode_categoricals(self, ordinal_cols: dict = None,
                             ohe_threshold: int = 10):
        """
        Encodes categorical columns:
        - Binary columns (2 unique values) → Label Encoding
        - Ordinal columns (ordered) → Ordinal Encoding with specified order
        - Low-cardinality nominal columns → One-Hot Encoding
        - High-cardinality nominal columns → Frequency Encoding

        Parameters:
            ordinal_cols:   Dict mapping column name to ordered category list.
                            e.g., {'Size': ['Small', 'Medium', 'Large']}
            ohe_threshold:  Max unique values for One-Hot Encoding; above = Frequency Encoding
        """
        cat_cols = self.df.select_dtypes(include=['object', 'category']).columns.tolist()

        for col in cat_cols:
            n_unique = self.df[col].nunique()

            if ordinal_cols and col in ordinal_cols:
                # Ordinal Encoding — preserves explicit order
                order = ordinal_cols[col]
                oe = OrdinalEncoder(categories=[order],
                                    handle_unknown='use_encoded_value',
                                    unknown_value=-1)
                self.df[[col]] = oe.fit_transform(self.df[[col]])
                self.log.append(f"encode_categoricals: OrdinalEncoding for '{col}'")

            elif n_unique == 2:
                # Binary column → simple Label Encoding (0/1)
                le = LabelEncoder()
                self.df[col] = le.fit_transform(self.df[col])
                self.label_encoders[col] = le
                self.log.append(f"encode_categoricals: LabelEncoding for '{col}'")

            elif n_unique <= ohe_threshold:
                # Low cardinality nominal → One-Hot Encoding
                dummies = pd.get_dummies(self.df[col], prefix=col, drop_first=True)
                # drop_first=True prevents multicollinearity (dummy variable trap)
                self.df = pd.concat([self.df.drop(columns=[col]), dummies], axis=1)
                self.log.append(f"encode_categoricals: OneHotEncoding for '{col}' ({n_unique} cats)")

            else:
                # High cardinality → Frequency Encoding (replace with proportion)
                freq_map = self.df[col].value_counts(normalize=True)
                self.df[col] = self.df[col].map(freq_map)
                self.log.append(f"encode_categoricals: FrequencyEncoding for '{col}' ({n_unique} cats)")

        return self


    # ------------------------------------------------------------------ #
    #  STEP 6: HANDLE DATE/TIME COLUMNS                                   #
    # ------------------------------------------------------------------ #
    def handle_datetime(self, datetime_cols: list = None,
                        reference_date: str = None,
                        drop_original: bool = True):
        """
        Parses datetime columns and extracts useful features.

        Parameters:
            datetime_cols:  List of column names containing datetime data.
                            If None, auto-detects columns with 'date' or 'time' in name.
            reference_date: A date string like '2024-01-01' to compute 'days_since' feature.
            drop_original:  Whether to drop the original datetime column after extraction.
        """
        if datetime_cols is None:
            # Auto-detect: columns whose name contains 'date' or 'time'
            datetime_cols = [
                col for col in self.df.columns
                if any(kw in col.lower() for kw in ['date', 'time', 'dt', 'timestamp'])
            ]

        ref = pd.Timestamp(reference_date) if reference_date else pd.Timestamp.now()

        for col in datetime_cols:
            try:
                self.df[col] = pd.to_datetime(self.df[col], errors='coerce')
                self.df[f'{col}_year']       = self.df[col].dt.year
                self.df[f'{col}_month']      = self.df[col].dt.month
                self.df[f'{col}_day']        = self.df[col].dt.day
                self.df[f'{col}_dayofweek']  = self.df[col].dt.dayofweek   # 0=Mon
                self.df[f'{col}_is_weekend'] = self.df[col].dt.dayofweek.isin([5, 6]).astype(int)
                self.df[f'{col}_quarter']    = self.df[col].dt.quarter
                self.df[f'{col}_days_since'] = (ref - self.df[col]).dt.days

                # Cyclic encoding for month (Jan and Dec are adjacent)
                self.df[f'{col}_month_sin'] = np.sin(2 * np.pi * self.df[f'{col}_month'] / 12)
                self.df[f'{col}_month_cos'] = np.cos(2 * np.pi * self.df[f'{col}_month'] / 12)

                if drop_original:
                    self.df.drop(columns=[col], inplace=True)

                self.log.append(f"handle_datetime: Extracted features from '{col}'")
            except Exception as e:
                self.log.append(f"handle_datetime: FAILED on '{col}' — {e}")

        return self


    # ------------------------------------------------------------------ #
    #  STEP 7: HANDLE SKEWED FEATURES                                     #
    # ------------------------------------------------------------------ #
    def fix_skewness(self, columns: list = None, threshold: float = 1.0):
        """
        Applies Yeo-Johnson transform to highly skewed numerical columns.

        Parameters:
            columns:   Columns to check. None = all numerical columns.
            threshold: Columns with |skew| > threshold will be transformed.
        """
        from sklearn.preprocessing import PowerTransformer

        if columns is None:
            columns = self.df.select_dtypes(include=[np.number]).columns.tolist()

        skewed = [col for col in columns if abs(self.df[col].skew()) > threshold]
        if skewed:
            pt = PowerTransformer(method='yeo-johnson', standardize=False)
            self.df[skewed] = pt.fit_transform(self.df[skewed])
            self.log.append(f"fix_skewness: Yeo-Johnson applied to {skewed}")
        else:
            self.log.append("fix_skewness: No significantly skewed columns found.")
        return self


    # ------------------------------------------------------------------ #
    #  STEP 8: SCALE NUMERICAL FEATURES                                   #
    # ------------------------------------------------------------------ #
    def scale_features(self, method: str = 'standard', columns: list = None):
        """
        Scales numerical columns to a uniform range/distribution.

        Parameters:
            method:  'standard' (z-score), 'minmax' (0 to 1), or 'robust' (IQR-based)
            columns: Columns to scale. None = all remaining numerical columns.
        """
        from sklearn.preprocessing import MinMaxScaler, RobustScaler

        if columns is None:
            columns = self.df.select_dtypes(include=[np.number]).columns.tolist()

        if method == 'standard':
            scaler = StandardScaler()
        elif method == 'minmax':
            scaler = MinMaxScaler()
        elif method == 'robust':
            scaler = RobustScaler()
        else:
            raise ValueError(f"Unknown scaling method: '{method}'. Choose 'standard', 'minmax', or 'robust'.")

        self.df[columns] = scaler.fit_transform(self.df[columns])
        self.scalers[method] = scaler
        self.log.append(f"scale_features: {method} scaling applied to {len(columns)} columns.")
        return self


    # ------------------------------------------------------------------ #
    #  STEP 9: PREPROCESS TEXT COLUMNS                                    #
    # ------------------------------------------------------------------ #
    def preprocess_text(self, text_cols: list, vectorize: bool = True,
                        max_features: int = 100):
        """
        Cleans text columns and optionally vectorizes them using TF-IDF.

        Parameters:
            text_cols:    List of column names containing text data.
            vectorize:    If True, replaces text column with TF-IDF numeric matrix.
            max_features: Max vocabulary size for TF-IDF.
        """
        try:
            import nltk
            from nltk.corpus import stopwords
            from nltk.stem import WordNetLemmatizer
            from sklearn.feature_extraction.text import TfidfVectorizer

            nltk.download('stopwords', quiet=True)
            nltk.download('wordnet', quiet=True)

            lemmatizer = WordNetLemmatizer()
            stop_words = set(stopwords.words('english'))

            def _clean(text):
                text = str(text).lower()
                text = re.sub(r'<.*?>', '', text)           # Remove HTML
                text = re.sub(r'http\S+|www\S+', '', text)  # Remove URLs
                text = re.sub(r'[^a-z\s]', '', text)        # Keep letters only
                text = re.sub(r'\s+', ' ', text).strip()
                tokens = text.split()
                tokens = [t for t in tokens if t not in stop_words]
                tokens = [lemmatizer.lemmatize(t) for t in tokens]
                return ' '.join(tokens)

            for col in text_cols:
                self.df[col] = self.df[col].apply(_clean)

                if vectorize:
                    tfidf = TfidfVectorizer(max_features=max_features)
                    tfidf_matrix = tfidf.fit_transform(self.df[col])
                    tfidf_df = pd.DataFrame(
                        tfidf_matrix.toarray(),
                        columns=[f'{col}_tfidf_{f}' for f in tfidf.get_feature_names_out()]
                    )
                    self.df = pd.concat(
                        [self.df.drop(columns=[col]), tfidf_df], axis=1
                    )

            self.log.append(f"preprocess_text: Processed {text_cols}, vectorize={vectorize}")

        except ImportError:
            self.log.append("preprocess_text: FAILED — nltk not installed. Run: pip install nltk")

        return self


    # ------------------------------------------------------------------ #
    #  UTILITY: PRINT SUMMARY                                             #
    # ------------------------------------------------------------------ #
    def summary(self):
        """Prints a summary of what was done and the shape change."""
        print("=" * 55)
        print("       PREPROCESSOR MANAGER — SUMMARY")
        print("=" * 55)
        print(f"  Original shape : {self.original_shape}")
        print(f"  Final shape    : {self.df.shape}")
        print(f"  Steps applied  : {len(self.log)}")
        print("-" * 55)
        for i, entry in enumerate(self.log, 1):
            print(f"  {i:2d}. {entry}")
        print("=" * 55)


    # ------------------------------------------------------------------ #
    #  UTILITY: GET RESULT                                                #
    # ------------------------------------------------------------------ #
    def get_processed_data(self) -> pd.DataFrame:
        """Returns the fully preprocessed DataFrame."""
        return self.df.copy()


    # ------------------------------------------------------------------ #
    #  ONE-SHOT: FULL PIPELINE                                            #
    # ------------------------------------------------------------------ #
    def run_full_pipeline(self,
                          num_cols_to_cast: list = None,
                          cat_cols_to_cast: list = None,
                          drop_duplicate_subset: list = None,
                          outlier_cols: list = None,
                          outlier_method: str = 'cap',
                          ordinal_cols: dict = None,
                          datetime_cols: list = None,
                          reference_date: str = None,
                          text_cols: list = None,
                          scaling_method: str = 'standard',
                          fix_skew: bool = True) -> pd.DataFrame:
        """
        Runs the complete preprocessing pipeline in the recommended order.

        Parameters:
            num_cols_to_cast:       Columns to cast as numeric
            cat_cols_to_cast:       Columns to cast as categorical strings
            drop_duplicate_subset:  Columns to check for duplicates (None = all)
            outlier_cols:           Numerical columns to handle outliers in
            outlier_method:         'cap' or 'remove'
            ordinal_cols:           Dict of {col: [ordered_categories]}
            datetime_cols:          Columns containing dates/timestamps
            reference_date:         Reference point for 'days_since' feature
            text_cols:              Columns containing free text
            scaling_method:         'standard', 'minmax', or 'robust'
            fix_skew:               Whether to apply skewness correction

        Returns:
            Preprocessed DataFrame
        """
        print("🚀 Starting PreprocessorManager Pipeline...")

        (self
         .fix_dtypes(num_cols=num_cols_to_cast, cat_cols=cat_cols_to_cast)
         .handle_missing_values()
         .remove_duplicates(subset=drop_duplicate_subset)
         .handle_outliers(columns=outlier_cols, method=outlier_method)
        )

        if datetime_cols:
            self.handle_datetime(datetime_cols=datetime_cols,
                                 reference_date=reference_date)

        if text_cols:
            self.preprocess_text(text_cols=text_cols)

        (self
         .encode_categoricals(ordinal_cols=ordinal_cols)
        )

        if fix_skew:
            self.fix_skewness()

        self.scale_features(method=scaling_method)
        self.summary()
        return self.get_processed_data()


# ============================================================ #
#  EXAMPLE USAGE
# ============================================================ #
if __name__ == "__main__":

    # Raw messy DataFrame simulating real-world data
    raw_data = pd.DataFrame({
        'Age':         [25, np.nan, 35, 40, 25, 150, 28, np.nan],
        'Salary':      [50000, 60000, np.nan, 80000, 50000, 55000, 62000, 47000],
        'Gender':      ['Male', 'Female', np.nan, 'Female', 'Male', 'Male', 'Other', 'Female'],
        'Department':  ['HR', 'IT', 'IT', 'Finance', 'HR', 'IT', 'Finance', 'HR'],
        'Experience':  ['Junior', 'Senior', 'Mid', 'Senior', 'Junior', 'Mid', 'Senior', 'Junior'],
        'Review':      ['Great place!', 'Not good at all.', 'Okay ish', 'Love it here!',
                        'Great place!', 'Meh.', 'Fantastic!!', 'Decent.'],
        'Join_Date':   ['2020-03-15', '2018-07-22', '2021-01-05', '2017-11-30',
                        '2020-03-15', '2019-06-10', '2022-02-28', '2020-09-01'],
    })

    print("Raw Data:")
    print(raw_data)
    print()

    pm = PreprocessorManager(raw_data)

    clean_df = pm.run_full_pipeline(
        outlier_cols=['Age', 'Salary'],
        outlier_method='cap',
        ordinal_cols={'Experience': ['Junior', 'Mid', 'Senior']},
        datetime_cols=['Join_Date'],
        reference_date='2024-01-01',
        text_cols=['Review'],
        scaling_method='standard',
        fix_skew=True
    )

    print("\n✅ Clean Preprocessed Data:")
    print(clean_df.head())
    print(f"\nFinal Shape: {clean_df.shape}")
```

---

## 📊 Preprocessing Order Cheat Sheet

```
Raw Data
   │
   ▼
1. Fix Data Types (casting)
   │
   ▼
2. Handle Missing Values (impute / drop)
   │
   ▼
3. Remove Duplicates
   │
   ▼
4. Handle Outliers (cap / remove)
   │
   ▼
5. Handle Datetime Columns (extract features)
   │
   ▼
6. Preprocess Text Columns (clean + vectorize)
   │
   ▼
7. Encode Categorical Variables (OHE / Ordinal / Frequency)
   │
   ▼
8. Fix Skewed Distributions (log / Yeo-Johnson)
   │
   ▼
9. Scale Numerical Features (StandardScaler / MinMax / Robust)
   │
   ▼
Clean, Model-Ready Data ✅
```

---

> **💡 Key Takeaway:** Preprocessing is not a one-time step — it's iterative. After each round of preprocessing, revisit your data, check distributions, check correlations, and refine. The `PreprocessorManager` class gives you a structured, reproducible, and customizable pipeline to do exactly that.
