# 🔍 Exploratory Data Analysis (EDA): A Complete Beginner's Guide

> *"EDA is detective work — you're looking for clues before you build the case."*  
> Before modeling, you must **understand** your data. EDA is how you do that.

---

## 📋 Table of Contents

1. [What is EDA?](#what-is-eda)
2. [EDA Before vs After Preprocessing](#eda-before-vs-after-preprocessing)
3. [Problems Solved by EDA](#problems-solved-by-eda)
4. [EDA Techniques](#eda-techniques)
   - [Shape & Data Types Inspection](#1-shape--data-types-inspection)
   - [Missing Value Analysis](#2-missing-value-analysis)
   - [Descriptive Statistics](#3-descriptive-statistics)
   - [Univariate Analysis — Numerical](#4-univariate-analysis--numerical)
   - [Univariate Analysis — Categorical](#5-univariate-analysis--categorical)
   - [Bivariate Analysis — Num vs Num](#6-bivariate-analysis--numerical-vs-numerical)
   - [Bivariate Analysis — Cat vs Num](#7-bivariate-analysis--categorical-vs-numerical)
   - [Bivariate Analysis — Cat vs Cat](#8-bivariate-analysis--categorical-vs-categorical)
   - [Correlation Analysis](#9-correlation-analysis)
   - [Outlier Detection & Visualization](#10-outlier-detection--visualization)
   - [Distribution & Skewness Analysis](#11-distribution--skewness-analysis)
   - [Target Variable Analysis](#12-target-variable-analysis)
   - [Feature Importance (Quick)](#13-feature-importance-quick)
   - [Time Series EDA](#14-time-series-eda)
   - [Multivariate Analysis](#15-multivariate-analysis)
5. [Generic EDAManager Class](#generic-edamanager-class)

---

## 📖 What is EDA?

Exploratory Data Analysis (EDA) is the process of **visually and statistically examining data** to understand its structure, patterns, anomalies, and relationships — before building any model.

EDA answers questions like:
- What does my data look like?
- Are there missing values? Duplicates? Outliers?
- How are features distributed?
- Which features are related to each other?
- Which features matter for the target variable?
- Is my data ready for modeling?

Think of it as the **map** you draw before navigating unknown territory.

---

## ⚖️ EDA Before vs After Preprocessing

EDA is done **twice** — each time with a different goal:

| Aspect | EDA Before Preprocessing | EDA After Preprocessing |
|--------|--------------------------|-------------------------|
| **Goal** | Understand raw data, spot problems | Verify fixes worked, confirm readiness |
| **Focus** | Missing values, outliers, data types | Clean distributions, balanced classes |
| **Distributions** | Often skewed, messy, extreme | Normalized, symmetric |
| **Correlations** | May be noisy or hidden | Cleaner, more reliable signals |
| **Categorical** | Raw strings, inconsistent labels | Encoded numerics |
| **Output** | Preprocessing plan | Modeling plan |

**Rule of Thumb:**
- EDA *before* preprocessing tells you **what to fix**.
- EDA *after* preprocessing confirms **the fix worked**.

---

## ⚠️ Problems Solved by EDA

| # | Problem | How EDA Reveals It | Solution Suggested |
|---|---------|-------------------|-------------------|
| 1 | **Hidden Missing Values** | Heatmaps, `.isnull()` counts | Impute or drop |
| 2 | **Wrong Data Types** | `.dtypes`, value inspection | Type cast |
| 3 | **Outliers** | Boxplots, Z-score plots | Cap, remove, or transform |
| 4 | **Skewed Distributions** | Histograms, skewness stats | Log / power transform |
| 5 | **Class Imbalance** | Target variable bar chart | Oversample, undersample, SMOTE |
| 6 | **Irrelevant Features** | Correlation heatmap, zero-variance check | Drop those features |
| 7 | **Multicollinearity** | Correlation matrix | Drop one of the correlated pair |
| 8 | **Inconsistent Categories** | `value_counts()` | Standardize labels |
| 9 | **Data Leakage** | Feature-target correlation analysis | Remove leaking features |
| 10 | **Duplicate Patterns** | Scatter plots, distribution plots | Remove duplicates |
| 11 | **Non-linear Relationships** | Scatter plots, pair plots | Use non-linear models or transformations |
| 12 | **Temporal Patterns** | Time series plots | Include time-based features |

---

## 🛠️ EDA Techniques

---

### 1. Shape & Data Types Inspection

#### 📌 What it does
Gives a first-level overview: how many rows/columns, what data types each column has, and how much memory it uses.

#### 🤔 Why it is done
Before anything else, you need to know: *What am I dealing with?* — how big is the data, do columns have the right type (a date stored as `object`? a number stored as `string`?).

#### 📂 On what types of data
Every dataset. Always the first step.

#### 💻 Implementation

```python
import pandas as pd
import numpy as np

df = pd.read_csv('your_data.csv')

# --- Basic Shape ---
print("Rows, Columns:", df.shape)        # e.g., (10000, 15)

# --- Column names ---
print("Columns:\n", df.columns.tolist())

# --- Data types per column ---
print("\nData Types:\n", df.dtypes)
# object  → string/mixed; should often be category or datetime
# int64   → integer
# float64 → decimal number
# bool    → True/False

# --- Memory usage ---
print("\nMemory Usage:")
print(df.memory_usage(deep=True).sum() / 1024**2, "MB")

# --- First and last 5 rows ---
print("\nFirst 5 rows:")
print(df.head())

print("\nLast 5 rows:")
print(df.tail())

# --- Random sample (avoids head-bias) ---
print("\nRandom Sample:")
print(df.sample(5, random_state=42))

# --- Count of each dtype ---
print("\nDtype Summary:")
print(df.dtypes.value_counts())
# float64    8
# object     4
# int64      3
```

**Explanation:**
- `df.shape` → `(rows, columns)` — gives dataset scale.
- `df.dtypes` → reveals type mismatches (dates stored as `object`, categories as `int`).
- `df.head()` → a quick sanity check but can be misleading for sorted data.
- `df.sample()` → better than head/tail since it shows random rows across the full dataset.
- `memory_usage(deep=True)` → important for large datasets; helps decide if you need to downcast types.

---

### 2. Missing Value Analysis

#### 📌 What it does
Counts, visualizes, and understands *where*, *how much*, and *why* data is missing.

#### 🤔 Why it is done
Missing data is not just a number — the *pattern* of missingness matters. If missing values cluster in one group, that's informative. Random missingness needs imputation; structured missingness might need a flag column.

#### 📂 On what types of data
All columns — numerical and categorical.

#### 💻 Implementation

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# --- Count and percentage missing ---
missing = pd.DataFrame({
    'Missing Count':   df.isnull().sum(),
    'Missing Percent': (df.isnull().sum() / len(df)) * 100
}).sort_values('Missing Percent', ascending=False)

print(missing[missing['Missing Count'] > 0])
#          Missing Count  Missing Percent
# Age              200           20.0
# Salary            50            5.0

# --- Heatmap of missing values (shows WHERE they appear) ---
plt.figure(figsize=(12, 6))
sns.heatmap(df.isnull(),
            cbar=False,          # No colour bar needed
            yticklabels=False,   # Row indices clutter the plot
            cmap='viridis')      # Yellow = missing, Purple = present
plt.title("Missing Value Heatmap")
plt.tight_layout()
plt.show()
# A column that's entirely yellow = almost all missing → consider dropping
# A horizontal yellow band = an entire row is missing → suspicious entry

# --- Bar chart of missing % ---
missing_pct = (df.isnull().sum() / len(df)) * 100
missing_pct = missing_pct[missing_pct > 0].sort_values(ascending=False)

plt.figure(figsize=(10, 5))
missing_pct.plot(kind='bar', color='tomato')
plt.title("Missing Value Percentage per Column")
plt.ylabel("% Missing")
plt.axhline(y=50, color='red', linestyle='--', label='50% threshold')
plt.legend()
plt.tight_layout()
plt.show()
# Columns above the red dashed line should likely be dropped entirely
```

**Explanation:**
- `isnull().sum()` → counts `NaN` per column; `/ len(df) * 100` converts to percentage.
- **Heatmap**: Yellow bands = missing values. A column that's all yellow = nearly useless. Horizontal yellow bands = entire row is blank — may be a data collection error.
- **50% rule**: Columns missing more than 50% of data are usually too sparse to reliably impute — they should be dropped.
- The *pattern* of missingness is as important as the count.

---

### 3. Descriptive Statistics

#### 📌 What it does
Summarizes each column with central tendency (mean, median) and spread (std, min, max, quartiles).

#### 🤔 Why it is done
Descriptive stats quickly flag anomalies: a minimum age of -5, a maximum salary of $1 billion, or a standard deviation that's larger than the mean all signal something wrong.

#### 📂 On what types of data
Primarily **numerical** columns. Separately check categorical summary.

#### 💻 Implementation

```python
import pandas as pd

# --- Numerical summary ---
print("Numerical Summary:")
print(df.describe().T)
# .T transposes for easier reading when many columns
#        count   mean    std    min    25%    50%    75%    max
# Age    990.0  34.2   10.5   18.0   26.0   33.0   42.0   95.0
# Salary 950.0  55000  20000  10000  40000  52000  68000  1000000

# Check for impossible values:
# Age max = 95? Plausible. min = 18? OK.
# Salary max = 1,000,000? Possible outlier.

# --- Custom stats to add ---
extra_stats = pd.DataFrame({
    'Skewness':  df.select_dtypes(include='number').skew(),
    'Kurtosis':  df.select_dtypes(include='number').kurtosis(),
    'Zeros':     (df.select_dtypes(include='number') == 0).sum(),
    'Negatives': (df.select_dtypes(include='number') < 0).sum()
})
print("\nExtra Stats:")
print(extra_stats)
# Skewness > 1  → right-skewed (log transform needed)
# Skewness < -1 → left-skewed
# Kurtosis > 3  → heavy tails (many outliers)
# Zeros count   → if many, might be missing data coded as 0

# --- Categorical summary ---
cat_cols = df.select_dtypes(include=['object', 'category']).columns
print("\nCategorical Summary:")
print(df[cat_cols].describe())
#        Gender Department
# count     990       1000
# unique      3          4
# top      Male         IT
# freq      480        350
```

**Explanation:**
- `describe()` → the most informative single-line summary in pandas.
- **Mean vs Median gap**: A large difference between `mean` and `50%` (median) signals skewness or outliers — outliers pull the mean.
- **Skewness**: > 1 or < -1 flags distributions that need transformation.
- **Kurtosis**: Very high kurtosis means heavy tails — many extreme values.
- **Zero count**: In columns like `Income`, zeros may be missing data coded incorrectly.

---

### 4. Univariate Analysis — Numerical

#### 📌 What it does
Examines the distribution of each individual numerical feature in isolation.

#### 🤔 Why it is done
Before comparing features to each other or to the target, you must understand each feature on its own: Is it normally distributed? Skewed? Bimodal? Are there extreme outliers?

#### 📂 On what types of data
**Numerical columns** (int, float).

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

num_cols = df.select_dtypes(include=np.number).columns.tolist()

fig, axes = plt.subplots(len(num_cols), 2,
                          figsize=(14, 4 * len(num_cols)))

for i, col in enumerate(num_cols):
    # --- Histogram (distribution shape) ---
    axes[i, 0].hist(df[col].dropna(), bins=30, color='steelblue',
                    edgecolor='white', alpha=0.8)
    axes[i, 0].axvline(df[col].mean(),   color='red',    linestyle='--', label='Mean')
    axes[i, 0].axvline(df[col].median(), color='orange', linestyle='--', label='Median')
    axes[i, 0].set_title(f'{col} — Histogram')
    axes[i, 0].set_xlabel(col)
    axes[i, 0].legend()

    # --- KDE Plot (smooth density curve) ---
    sns.kdeplot(df[col].dropna(), ax=axes[i, 1], fill=True, color='steelblue')
    axes[i, 1].set_title(f'{col} — KDE (Density)')
    axes[i, 1].set_xlabel(col)

plt.tight_layout()
plt.show()

# What to look for:
# - Bell shape       → Normally distributed (good for linear models)
# - Long right tail  → Positive skew (apply log transform)
# - Long left tail   → Negative skew
# - Two peaks        → Bimodal (two sub-populations; consider splitting)
# - Spike at one val → Possible encoding of missing as 0 or -1
```

**Explanation:**
- **Histogram**: Shows frequency of value ranges (bins). The shape reveals distribution type.
- **Mean vs Median lines**: If mean >> median, right skew (pulled by large values). If mean << median, left skew.
- **KDE (Kernel Density Estimate)**: A smooth, continuous version of the histogram — better for comparing shapes.
- **Bimodal distribution** (two humps): Often means your data is actually two separate populations mixed together — worth investigating if there's a hidden grouping variable.

---

### 5. Univariate Analysis — Categorical

#### 📌 What it does
Counts and visualizes the frequency of each category within categorical columns.

#### 🤔 Why it is done
To spot: dominant categories (one category covers 90% of rows), rare categories (may need grouping), inconsistent labels (`"Male"` vs `"male"` vs `"M"`), and unexpected categories.

#### 📂 On what types of data
**Categorical / object / string columns.**

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns

cat_cols = df.select_dtypes(include=['object', 'category']).columns.tolist()

for col in cat_cols:
    vc = df[col].value_counts()
    pct = df[col].value_counts(normalize=True) * 100

    print(f"\n{'='*40}")
    print(f"Column: {col}  |  Unique: {df[col].nunique()}")
    print(pd.concat([vc, pct.round(2)], axis=1,
                    keys=['Count', 'Percent']))

    # Bar chart
    plt.figure(figsize=(8, 4))
    sns.countplot(data=df, x=col,
                  order=vc.index,          # Order by frequency
                  palette='Set2')
    plt.title(f'{col} — Value Counts')
    plt.xticks(rotation=45, ha='right')
    plt.tight_layout()
    plt.show()

# What to look for:
# - One category >> 80% → dominant; model may just predict that category
# - Category < 1%       → rare; may need to group into "Other"
# - Near-duplicate labels → 'male', 'Male', 'MALE' all appear → standardize
# - Unexpected values   → 'N/A', '?', 'Unknown' coded as categories
```

**Explanation:**
- `value_counts()` → counts occurrences of each unique value.
- `normalize=True` → shows proportions (0–1); multiply by 100 for percentages.
- **Ordering bars by frequency** makes it easy to spot dominant vs rare categories.
- **Rare categories**: If `"Other"` is 0.01% of data, a model will rarely see it — consider merging similar rare categories.

---

### 6. Bivariate Analysis — Numerical vs Numerical

#### 📌 What it does
Examines the relationship between two numerical variables.

#### 🤔 Why it is done
To detect: linear or non-linear relationships, correlated features (potential multicollinearity), and features that have predictive power for the target.

#### 📂 On what types of data
**Two numerical columns.**

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

# --- Scatter Plot (raw relationship) ---
plt.figure(figsize=(7, 5))
sns.scatterplot(data=df, x='Age', y='Salary', alpha=0.5, color='steelblue')
plt.title("Age vs Salary")
plt.show()
# A diagonal cloud → linear relationship (good for linear regression)
# A curved cloud   → non-linear (need polynomial features or tree models)
# A horizontal cloud → no relationship (feature may not be useful)
# Outliers appear as points far from the main cloud

# --- Scatter with regression line ---
plt.figure(figsize=(7, 5))
sns.regplot(data=df, x='Age', y='Salary',
            scatter_kws={'alpha': 0.4},
            line_kws={'color': 'red'})
plt.title("Age vs Salary with Regression Line")
plt.show()

# --- Pair Plot (all numerical pairs at once) ---
num_df = df.select_dtypes(include=np.number).iloc[:, :6]  # First 6 columns max
sns.pairplot(num_df, diag_kind='kde', plot_kws={'alpha': 0.5})
plt.suptitle("Pair Plot — All Numerical Features", y=1.02)
plt.show()
# Diagonal: Distribution of each feature
# Off-diagonal: Scatter plot between each pair
# Look for: ellipse-like clouds (correlation), no-pattern (independence)

# --- Pearson Correlation (linear relationship strength) ---
corr = df[['Age', 'Salary', 'YearsExp']].corr()
print(corr)
# Values range -1 to +1
# +1 → perfect positive linear relationship
#  0 → no linear relationship
# -1 → perfect negative linear relationship
```

**Explanation:**
- **Scatter plot**: The most direct way to see how two variables move together.
- **regplot**: Adds a fitted line with confidence band — positive slope = positive correlation.
- **Pair plot**: A grid of all pairwise scatter plots. The diagonal shows each feature's own distribution. Very efficient for seeing all relationships at once.
- **Pearson correlation**: Measures linear association. Only measures *linear* — a strong curved relationship may show low Pearson r.

---

### 7. Bivariate Analysis — Categorical vs Numerical

#### 📌 What it does
Shows how the distribution of a numerical variable differs across categories.

#### 🤔 Why it is done
To understand whether a category is a meaningful predictor of a numerical outcome — e.g., does `Department` significantly affect `Salary`?

#### 📂 On what types of data
One **categorical** column and one **numerical** column.

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns

# --- Box Plot (distribution per category) ---
plt.figure(figsize=(9, 5))
sns.boxplot(data=df, x='Department', y='Salary', palette='Set2')
plt.title("Salary Distribution by Department")
plt.xticks(rotation=30)
plt.show()
# Box = IQR (25th to 75th percentile)
# Line inside box = Median
# Whiskers = range excluding outliers
# Dots beyond whiskers = outliers
# Comparison: If boxes are at very different heights → category IS predictive

# --- Violin Plot (combines box + KDE for richer shape info) ---
plt.figure(figsize=(9, 5))
sns.violinplot(data=df, x='Department', y='Salary',
               palette='pastel', inner='quartile')
plt.title("Salary Distribution by Department (Violin)")
plt.xticks(rotation=30)
plt.show()
# Wider violin section = more data concentrated there
# Bimodal violin = two sub-groups within that category

# --- Group Means Bar Chart ---
group_means = df.groupby('Department')['Salary'].mean().sort_values(ascending=False)
plt.figure(figsize=(8, 4))
group_means.plot(kind='bar', color='steelblue', edgecolor='white')
plt.title("Mean Salary by Department")
plt.ylabel("Mean Salary")
plt.xticks(rotation=30)
plt.tight_layout()
plt.show()
```

**Explanation:**
- **Box plot**: Quickly shows if medians differ between groups and how spread-out each group is. Overlapping boxes = weak relationship; well-separated boxes = strong relationship.
- **Violin plot**: Wider = more data at that value. Better than a box plot for seeing if a group has a bimodal distribution.
- **Group means**: Simplest check — if means are very different across categories, the category has predictive power.

---

### 8. Bivariate Analysis — Categorical vs Categorical

#### 📌 What it does
Examines the relationship or association between two categorical variables.

#### 🤔 Why it is done
To detect dependencies between categories — e.g., is `Gender` associated with `Department`? Are people in one department more likely to be of a certain experience level?

#### 📂 On what types of data
Two **categorical** columns.

#### 💻 Implementation

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import chi2_contingency

# --- Contingency / Cross-Tab Table ---
ct = pd.crosstab(df['Department'], df['Gender'])
print("Contingency Table:")
print(ct)
#             Female  Male  Other
# Department
# Finance        120   140      5
# HR             180    90      8
# IT              60   250     12

# --- Normalize to see proportions within each department ---
ct_pct = pd.crosstab(df['Department'], df['Gender'], normalize='index') * 100
print("\nRow Percentages:")
print(ct_pct.round(1))

# --- Heatmap of contingency table ---
plt.figure(figsize=(8, 5))
sns.heatmap(ct, annot=True, fmt='d', cmap='Blues')
plt.title("Department vs Gender Count")
plt.show()
# Darker cells = higher count; annotation shows exact count

# --- Stacked Bar Chart ---
ct_pct.plot(kind='bar', stacked=True, figsize=(9, 5), colormap='Set2')
plt.title("Gender Distribution within Departments")
plt.ylabel("Percentage (%)")
plt.xticks(rotation=30)
plt.legend(title='Gender')
plt.tight_layout()
plt.show()

# --- Chi-Square Test for statistical independence ---
chi2, p_value, dof, expected = chi2_contingency(ct)
print(f"\nChi2 = {chi2:.2f}, p-value = {p_value:.4f}")
if p_value < 0.05:
    print("✅ Statistically DEPENDENT — Department and Gender ARE related.")
else:
    print("❌ Statistically INDEPENDENT — No significant relationship.")
# p < 0.05 → reject null hypothesis (that they are independent)
```

**Explanation:**
- **Cross-tab**: A frequency table showing how many observations fall in each combination of categories.
- **Row normalization**: Converts counts to percentages within each row — makes comparisons fair even when group sizes differ.
- **Stacked bar**: Visual version of the cross-tab — easy to see the composition of each category.
- **Chi-Square test**: A statistical test. Low p-value (< 0.05) means the association is statistically significant, not just random chance.

---

### 9. Correlation Analysis

#### 📌 What it does
Measures how strongly and in what direction pairs of numerical features are linearly related.

#### 🤔 Why it is done
To detect:
1. **Predictive features**: Features highly correlated with the target.
2. **Multicollinearity**: Two features highly correlated with *each other* — keeping both adds noise.

#### 📂 On what types of data
**Numerical columns only.** Use Chi-Square (Technique 8) for categorical.

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

num_df = df.select_dtypes(include=np.number)

# --- Pearson Correlation Matrix ---
corr_matrix = num_df.corr(method='pearson')
# 'pearson'  → linear correlation (default)
# 'spearman' → rank-based; robust to outliers and non-linear monotonic relationships
# 'kendall'  → another rank-based; better for small samples

# --- Heatmap ---
plt.figure(figsize=(12, 10))
mask = np.triu(np.ones_like(corr_matrix, dtype=bool))  # Show only lower triangle
sns.heatmap(corr_matrix,
            annot=True,       # Show correlation values in each cell
            fmt='.2f',        # 2 decimal places
            mask=mask,        # Hide upper triangle (it's a mirror)
            cmap='coolwarm',  # Red=positive, Blue=negative
            center=0,         # Colour anchored at 0
            linewidths=0.5,
            vmin=-1, vmax=1)
plt.title("Pearson Correlation Heatmap")
plt.tight_layout()
plt.show()

# --- Find highly correlated feature pairs (multicollinearity risk) ---
threshold = 0.85
high_corr_pairs = []
for i in range(len(corr_matrix.columns)):
    for j in range(i):
        if abs(corr_matrix.iloc[i, j]) > threshold:
            high_corr_pairs.append({
                'Feature 1': corr_matrix.columns[i],
                'Feature 2': corr_matrix.columns[j],
                'Correlation': round(corr_matrix.iloc[i, j], 3)
            })
print("High Correlation Pairs (|r| > 0.85):")
print(pd.DataFrame(high_corr_pairs))

# --- Target correlation bar chart ---
if 'Target' in df.columns:
    target_corr = corr_matrix['Target'].drop('Target').sort_values(key=abs, ascending=False)
    plt.figure(figsize=(8, 5))
    target_corr.plot(kind='bar',
                     color=['green' if v > 0 else 'red' for v in target_corr])
    plt.title("Feature Correlation with Target")
    plt.axhline(0, color='black', linewidth=0.8)
    plt.ylabel("Pearson r")
    plt.tight_layout()
    plt.show()
```

**Explanation:**
- **Correlation matrix**: Symmetric matrix where each cell is the Pearson r between two features. The diagonal is always 1.0 (a feature perfectly correlates with itself).
- `mask=np.triu(...)` → hides the upper triangle since it's a mirror of the lower half — reduces visual clutter.
- **Red cells (positive)**: Both features increase together.
- **Blue cells (negative)**: One increases as the other decreases.
- **Multicollinearity**: If two features have `|r| > 0.85`, they carry nearly the same information. Drop one.
- **Spearman** correlation is better when data isn't normally distributed or has outliers — it's based on ranks, not raw values.

---

### 10. Outlier Detection & Visualization

#### 📌 What it does
Identifies data points that fall far outside the typical range of a feature, both visually and numerically.

#### 🤔 Why it is done
Outliers distort statistics and confuse ML models. But not all outliers should be removed — some are real (a billionaire in a salary dataset) and some are data errors (age = 999). EDA helps you distinguish.

#### 📂 On what types of data
**Numerical columns.**

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from scipy import stats

num_cols = df.select_dtypes(include=np.number).columns.tolist()

# --- Box Plots (visual outlier detection) ---
fig, axes = plt.subplots(1, len(num_cols), figsize=(4 * len(num_cols), 5))
for i, col in enumerate(num_cols):
    axes[i].boxplot(df[col].dropna(), vert=True, patch_artist=True,
                    boxprops=dict(facecolor='steelblue', alpha=0.6))
    axes[i].set_title(col)
    axes[i].set_xlabel('')
plt.suptitle("Box Plots — Outlier Detection")
plt.tight_layout()
plt.show()
# Points above/below the whiskers = outliers by IQR definition

# --- IQR-based count ---
print("Outlier Counts (IQR Method):")
for col in num_cols:
    Q1  = df[col].quantile(0.25)
    Q3  = df[col].quantile(0.75)
    IQR = Q3 - Q1
    outliers = df[(df[col] < Q1 - 1.5 * IQR) | (df[col] > Q3 + 1.5 * IQR)]
    pct = len(outliers) / len(df) * 100
    print(f"  {col}: {len(outliers)} outliers ({pct:.1f}%)")

# --- Z-Score method ---
print("\nOutlier Counts (Z-Score Method, |z| > 3):")
for col in num_cols:
    z_scores = np.abs(stats.zscore(df[col].dropna()))
    n_outliers = (z_scores > 3).sum()
    print(f"  {col}: {n_outliers} outliers")

# --- Scatter plot: spot multi-dimensional outliers ---
plt.figure(figsize=(7, 5))
plt.scatter(df['Age'], df['Salary'], alpha=0.5, color='steelblue', label='Normal')

# Highlight IQR outliers in red
q1, q3 = df['Salary'].quantile([0.25, 0.75])
iqr = q3 - q1
outlier_mask = (df['Salary'] < q1 - 1.5*iqr) | (df['Salary'] > q3 + 1.5*iqr)
plt.scatter(df.loc[outlier_mask, 'Age'], df.loc[outlier_mask, 'Salary'],
            color='red', label='Outlier (IQR)', zorder=5)
plt.title("Age vs Salary — Outliers Highlighted")
plt.legend()
plt.show()
```

**Explanation:**
- **Box plot**: Whiskers extend to 1.5 × IQR from Q1/Q3. Any point beyond whiskers is flagged as outlier.
- **IQR method**: Rule-of-thumb, no distribution assumption. Robust for skewed data.
- **Z-Score method**: Assumes normal distribution. |z| > 3 means the value is 3 standard deviations away — very unusual.
- **Red highlighting**: Visually separating outlier points in a scatter plot helps understand whether they form a separate cluster or are truly isolated errors.

---

### 11. Distribution & Skewness Analysis

#### 📌 What it does
Quantifies the shape of each feature's distribution and identifies how far it deviates from a normal (bell-curve) shape.

#### 🤔 Why it is done
Many models (Linear Regression, Logistic Regression, LDA) assume features are roughly normally distributed. High skewness degrades their performance. This analysis tells you which columns need transformation.

#### 📂 On what types of data
**Numerical columns.**

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from scipy import stats

num_cols = df.select_dtypes(include=np.number).columns.tolist()

# --- Skewness and Kurtosis table ---
shape_stats = pd.DataFrame({
    'Skewness': df[num_cols].skew(),
    'Kurtosis': df[num_cols].kurtosis()
}).round(3)
shape_stats['Skew Flag'] = shape_stats['Skewness'].apply(
    lambda x: '⚠️ High Right Skew' if x > 1
              else ('⚠️ High Left Skew' if x < -1 else '✅ OK')
)
print(shape_stats)

# --- QQ Plot (compare to theoretical normal) ---
fig, axes = plt.subplots(1, len(num_cols), figsize=(5 * len(num_cols), 4))
for i, col in enumerate(num_cols):
    stats.probplot(df[col].dropna(), dist='norm', plot=axes[i])
    axes[i].set_title(f'{col} — QQ Plot')
plt.tight_layout()
plt.show()
# Perfect normal → all dots on the diagonal line
# S-curve       → heavy tails (outliers)
# Curved up     → right skew
# Curved down   → left skew

# --- Before vs After Log Transform comparison ---
col = 'Salary'
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

ax1.hist(df[col].dropna(), bins=30, color='tomato', edgecolor='white')
ax1.set_title(f'{col} — Before (Skew: {df[col].skew():.2f})')

log_col = np.log1p(df[col].dropna())
ax2.hist(log_col, bins=30, color='seagreen', edgecolor='white')
ax2.set_title(f'log({col}) — After (Skew: {log_col.skew():.2f})')

plt.suptitle("Log Transform Effect on Skewness")
plt.tight_layout()
plt.show()
```

**Explanation:**
- **Skewness**: > 1 = right-skewed (long tail to the right, e.g., income, house prices). < -1 = left-skewed. Between -1 and 1 = acceptable.
- **Kurtosis**: > 3 = leptokurtic (heavy tails, more outliers than normal). < 3 = platykurtic (light tails, fewer extremes).
- **QQ Plot**: Plots quantiles of your data against quantiles of a normal distribution. If points follow the diagonal line, data is normal. Deviations from the line show where and how it diverges.
- **Before/After**: Always verify a transform *actually reduced skewness* — don't assume.

---

### 12. Target Variable Analysis

#### 📌 What it does
Specifically examines the distribution and characteristics of the variable you're trying to predict (the label/target).

#### 🤔 Why it is done
Everything flows from the target:
- Regression target: Is it skewed? Does it need log transform?
- Classification target: Is it imbalanced? This changes which metrics and algorithms to use.

#### 📂 On what types of data
The **target column** — numerical (regression) or categorical (classification).

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

target = 'Salary'       # Change to your target column
target_type = 'numerical'   # or 'categorical'

if target_type == 'numerical':
    fig, axes = plt.subplots(1, 3, figsize=(16, 4))

    # Distribution
    axes[0].hist(df[target].dropna(), bins=30, color='steelblue', edgecolor='white')
    axes[0].axvline(df[target].mean(),   color='red',    linestyle='--', label='Mean')
    axes[0].axvline(df[target].median(), color='orange', linestyle='--', label='Median')
    axes[0].set_title(f'{target} Distribution')
    axes[0].legend()

    # Box plot
    axes[1].boxplot(df[target].dropna(), vert=True, patch_artist=True,
                    boxprops=dict(facecolor='steelblue', alpha=0.6))
    axes[1].set_title(f'{target} Box Plot')

    # QQ Plot
    from scipy import stats
    stats.probplot(df[target].dropna(), dist='norm', plot=axes[2])
    axes[2].set_title(f'{target} QQ Plot')

    plt.suptitle(f"Target Variable: {target}")
    plt.tight_layout()
    plt.show()

    print(f"\nTarget Stats:\n{df[target].describe()}")
    print(f"Skewness: {df[target].skew():.3f}")

elif target_type == 'categorical':
    vc = df[target].value_counts()
    pct = df[target].value_counts(normalize=True) * 100

    print("Class Distribution:")
    print(pd.concat([vc, pct.round(2)], axis=1, keys=['Count', 'Percent']))

    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

    # Count bar
    ax1.bar(vc.index.astype(str), vc.values, color='steelblue', edgecolor='white')
    ax1.set_title(f'{target} — Class Counts')
    ax1.set_ylabel('Count')

    # Proportion pie
    ax2.pie(pct.values, labels=pct.index.astype(str), autopct='%1.1f%%',
            colors=sns.color_palette('Set2'))
    ax2.set_title(f'{target} — Class Proportions')

    plt.tight_layout()
    plt.show()

    # Imbalance ratio
    majority = vc.max()
    minority = vc.min()
    print(f"\nImbalance Ratio: {majority/minority:.1f}:1")
    if majority/minority > 5:
        print("⚠️  Severe class imbalance — use SMOTE, class weights, or resampling!")
```

**Explanation:**
- For **regression targets**: Check if the target itself is skewed — log-transforming the target often improves regression model performance significantly.
- For **classification targets**: The imbalance ratio is critical. A 100:1 ratio means the model will almost always predict the majority class — you need to address this.
- The **mean vs median gap** in the target tells you if a few extreme outcomes are pulling predictions.

---

### 13. Feature Importance (Quick)

#### 📌 What it does
Provides a fast estimate of which features are likely most useful for predicting the target, before modeling.

#### 🤔 Why it is done
With many features, you need a quick way to prioritize which ones are worth deeper analysis and which can likely be dropped to reduce noise.

#### 📂 On what types of data
All encoded numerical features + a target variable.

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import numpy as np
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
from sklearn.preprocessing import LabelEncoder

# Prepare data — drop rows with NaN for this quick check
df_clean = df.dropna()
X = df_clean.drop(columns=['Target'])
y = df_clean['Target']

# Encode any remaining categoricals simply for this quick check
for col in X.select_dtypes(include='object').columns:
    X[col] = LabelEncoder().fit_transform(X[col].astype(str))

# Choose model based on target type
if y.dtype == 'object' or y.nunique() < 15:
    model = RandomForestClassifier(n_estimators=100, random_state=42)
else:
    model = RandomForestRegressor(n_estimators=100, random_state=42)

model.fit(X, y)

# --- Feature Importance ---
importance = pd.Series(model.feature_importances_, index=X.columns)
importance = importance.sort_values(ascending=True)

plt.figure(figsize=(8, max(5, len(importance) * 0.35)))
importance.plot(kind='barh', color='steelblue', edgecolor='white')
plt.title("Feature Importance (Random Forest)")
plt.xlabel("Importance Score")
plt.axvline(importance.mean(), color='red', linestyle='--', label='Mean Importance')
plt.legend()
plt.tight_layout()
plt.show()

# Print top features
print("Top 10 Features:")
print(importance.sort_values(ascending=False).head(10))
# Features with near-zero importance → likely irrelevant; consider dropping
```

**Explanation:**
- **Random Forest importance**: Each feature's importance is measured by how much it reduces prediction error when used for splitting across all trees.
- Features with **near-zero importance** are candidates for removal — they add computational cost but no predictive value.
- This is a **quick heuristic**, not a definitive answer. Use SHAP values for more rigorous feature importance.
- This step comes *after* basic encoding to make the dataset model-readable, but it's still exploratory — not the final model.

---

### 14. Time Series EDA

#### 📌 What it does
Examines patterns in data that is indexed or ordered by time — trends, seasonality, cyclic patterns, and anomalies over time.

#### 🤔 Why it is done
Time-ordered data has temporal dependencies — today's value depends on yesterday's. Ignoring time structure leads to data leakage and bad predictions.

#### 📂 On what types of data
Data with a **datetime column** or time-indexed DataFrames.

#### 💻 Implementation

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df['Date'] = pd.to_datetime(df['Date'])
df.set_index('Date', inplace=True)

# --- Line Plot (overall trend) ---
plt.figure(figsize=(14, 5))
df['Sales'].plot(color='steelblue', linewidth=1)
plt.title("Sales Over Time")
plt.ylabel("Sales")
plt.show()
# Look for: upward/downward trend, cyclic patterns, sudden spikes/dips

# --- Rolling Average (smooth out noise) ---
plt.figure(figsize=(14, 5))
df['Sales'].plot(color='lightblue', alpha=0.5, label='Daily')
df['Sales'].rolling(window=7).mean().plot(color='steelblue', label='7-Day Rolling Avg')
df['Sales'].rolling(window=30).mean().plot(color='red', linestyle='--', label='30-Day Rolling Avg')
plt.title("Sales with Rolling Averages")
plt.legend()
plt.show()

# --- Seasonality: Monthly and Weekly Patterns ---
df_reset = df.reset_index()
df_reset['Month']   = df_reset['Date'].dt.month
df_reset['Weekday'] = df_reset['Date'].dt.day_name()

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

# Monthly
monthly = df_reset.groupby('Month')['Sales'].mean()
ax1.bar(monthly.index, monthly.values, color='steelblue', edgecolor='white')
ax1.set_title("Avg Sales by Month (Seasonality)")
ax1.set_xlabel("Month")

# Day of week
weekday_order = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']
weekly = df_reset.groupby('Weekday')['Sales'].mean().reindex(weekday_order)
ax2.bar(weekly.index, weekly.values, color='seagreen', edgecolor='white')
ax2.set_title("Avg Sales by Weekday")
ax2.set_xticklabels(weekday_order, rotation=30)
plt.tight_layout()
plt.show()

# --- Year-over-Year Comparison ---
df_reset['Year'] = df_reset['Date'].dt.year
plt.figure(figsize=(12, 5))
for yr in df_reset['Year'].unique():
    subset = df_reset[df_reset['Year'] == yr]
    plt.plot(subset['Month'], subset.groupby('Month')['Sales'].mean(),
             marker='o', label=str(yr))
plt.title("Year-over-Year Monthly Sales Comparison")
plt.legend(title='Year')
plt.xlabel("Month")
plt.ylabel("Avg Sales")
plt.show()
```

**Explanation:**
- **Line plot**: The foundational time series chart. Trend direction, sudden changes (spikes = promotions? dips = outages?), and seasonality are all visible.
- **Rolling average**: Smooths short-term noise to reveal the underlying trend. 7-day captures weekly patterns; 30-day shows monthly trend.
- **Monthly seasonality**: Does December always spike? Do summers always dip? These patterns must be captured in features.
- **Year-over-Year**: Checks if the same month performs similarly across years — confirms seasonality or shows year-specific anomalies.

---

### 15. Multivariate Analysis

#### 📌 What it does
Examines relationships among three or more variables simultaneously.

#### 🤔 Why it is done
Real-world data has interactions — the effect of `Age` on `Salary` may differ by `Gender`. Single-variable or two-variable analysis misses these interactions.

#### 📂 On what types of data
Mix of numerical and categorical columns.

#### 💻 Implementation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

# --- Scatter plot with 3rd variable as colour (hue) ---
plt.figure(figsize=(9, 6))
sns.scatterplot(data=df, x='Age', y='Salary',
                hue='Department',   # 3rd variable as colour
                size='YearsExp',    # 4th variable as point size
                palette='Set1',
                alpha=0.7)
plt.title("Age vs Salary by Department and Experience")
plt.legend(bbox_to_anchor=(1, 1))
plt.tight_layout()
plt.show()
# Each colour = a Department; each size = Years of Experience
# You can spot: does IT earn more than HR at the same age?

# --- FacetGrid: Same plot, split by category ---
g = sns.FacetGrid(df, col='Department', col_wrap=3, height=4)
g.map(sns.scatterplot, 'Age', 'Salary', alpha=0.5, color='steelblue')
g.set_axis_labels("Age", "Salary")
g.set_titles("{col_name}")
g.figure.suptitle("Age vs Salary within Each Department", y=1.02)
plt.tight_layout()
plt.show()
# Each panel = one department; compare slope/pattern across departments

# --- Grouped Boxplot (2 categorical + 1 numerical) ---
plt.figure(figsize=(12, 5))
sns.boxplot(data=df, x='Department', y='Salary',
            hue='Gender', palette='Set2')
plt.title("Salary by Department and Gender")
plt.legend(title='Gender', bbox_to_anchor=(1, 1))
plt.xticks(rotation=30)
plt.tight_layout()
plt.show()

# --- Correlation Heatmap by Subgroup ---
for dept in df['Department'].unique():
    subset = df[df['Department'] == dept].select_dtypes(include=np.number)
    print(f"\n{dept} Correlation:")
    print(subset.corr().round(2))
```

**Explanation:**
- **Scatter with `hue` and `size`**: Adds two extra dimensions to a 2D plot — colour = 3rd variable, size = 4th variable. Very information-dense.
- **FacetGrid**: Creates a grid of identical plots, one per category value. This makes it easy to compare whether patterns are consistent or differ across departments/groups.
- **Grouped boxplot**: `hue='Gender'` adds a split inside each department's box — you can see salary distribution by *both* department and gender simultaneously.
- **Subgroup correlation**: Correlations may differ within subgroups. Overall correlation might be 0.3 but within `IT` it might be 0.8 — a phenomenon called Simpson's Paradox.

---

---

## 🏗️ Generic EDAManager Class

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
from scipy.stats import chi2_contingency
import warnings
warnings.filterwarnings('ignore')

# Set a clean visual style for all plots
sns.set_theme(style='whitegrid', palette='Set2')
plt.rcParams.update({'figure.dpi': 100, 'font.size': 11})


class EDAManager:
    """
    A Generic Exploratory Data Analysis (EDA) Manager.

    Performs structured EDA for any real-world tabular dataset.
    Works both BEFORE preprocessing (to find problems) and
    AFTER preprocessing (to verify fixes).

    Usage:
        eda = EDAManager(df, target='Salary', target_type='numerical')

        # Run individual analyses:
        eda.inspect_shape()
        eda.analyze_missing_values()
        eda.descriptive_statistics()
        eda.univariate_analysis()
        eda.correlation_analysis()
        eda.analyze_target()

        # Or run everything at once:
        eda.run_full_eda()
    """

    def __init__(self, df: pd.DataFrame,
                 target: str = None,
                 target_type: str = 'auto'):
        """
        Initialize EDAManager.

        Parameters:
            df:          The raw or preprocessed DataFrame to analyze.
            target:      Name of the target/label column (optional).
            target_type: 'numerical', 'categorical', or 'auto' (auto-detected).
        """
        self.df = df.copy()
        self.target = target
        self.n_rows, self.n_cols = df.shape
        self.log = []

        # Separate features from target if provided
        self.feature_df = df.drop(columns=[target]) if target else df
        self.num_cols = self.feature_df.select_dtypes(include=np.number).columns.tolist()
        self.cat_cols = self.feature_df.select_dtypes(
                            include=['object', 'category']).columns.tolist()

        # Auto-detect target type
        if target and target_type == 'auto':
            if df[target].dtype == 'object' or df[target].nunique() < 15:
                self.target_type = 'categorical'
            else:
                self.target_type = 'numerical'
        else:
            self.target_type = target_type

        print(f"✅ EDAManager initialized")
        print(f"   Shape: {self.n_rows} rows × {self.n_cols} columns")
        print(f"   Numerical: {len(self.num_cols)} cols | Categorical: {len(self.cat_cols)} cols")
        if target:
            print(f"   Target: '{target}' ({self.target_type})")


    # ------------------------------------------------------------------ #
    #  1. SHAPE & DATA TYPES                                              #
    # ------------------------------------------------------------------ #
    def inspect_shape(self):
        """Prints shape, dtypes, memory usage, and data samples."""
        print("\n" + "="*60)
        print("  1. SHAPE & DATA TYPE INSPECTION")
        print("="*60)

        print(f"\n📐 Shape: {self.df.shape[0]} rows × {self.df.shape[1]} columns")
        print(f"💾 Memory: {self.df.memory_usage(deep=True).sum() / 1024**2:.2f} MB")

        print("\n📊 Data Types:")
        dtype_summary = self.df.dtypes.value_counts().reset_index()
        dtype_summary.columns = ['dtype', 'count']
        print(dtype_summary.to_string(index=False))

        print("\n📋 Column Details:")
        col_info = pd.DataFrame({
            'dtype':    self.df.dtypes,
            'non_null': self.df.notnull().sum(),
            'null':     self.df.isnull().sum(),
            'unique':   self.df.nunique(),
            'sample':   self.df.iloc[0]
        })
        print(col_info.to_string())

        print("\n🔍 Random Sample (5 rows):")
        print(self.df.sample(min(5, len(self.df)), random_state=42).to_string())

        self.log.append("inspect_shape: ✅ Done")


    # ------------------------------------------------------------------ #
    #  2. MISSING VALUE ANALYSIS                                          #
    # ------------------------------------------------------------------ #
    def analyze_missing_values(self):
        """Visualizes and summarizes missing values across all columns."""
        print("\n" + "="*60)
        print("  2. MISSING VALUE ANALYSIS")
        print("="*60)

        missing = pd.DataFrame({
            'Count':   self.df.isnull().sum(),
            'Percent': (self.df.isnull().sum() / len(self.df)) * 100
        }).sort_values('Percent', ascending=False)
        missing = missing[missing['Count'] > 0]

        if missing.empty:
            print("✅ No missing values found!")
            self.log.append("analyze_missing_values: No missing values.")
            return

        print(f"\n⚠️  {len(missing)} columns have missing values:")
        print(missing.round(2).to_string())

        fig, axes = plt.subplots(1, 2, figsize=(14, 5))

        # Heatmap
        sns.heatmap(self.df.isnull(), cbar=False, yticklabels=False,
                    cmap='viridis', ax=axes[0])
        axes[0].set_title("Missing Value Heatmap\n(Yellow = Missing)")

        # Bar chart
        missing['Percent'].plot(kind='bar', color='tomato',
                                edgecolor='white', ax=axes[1])
        axes[1].axhline(50, color='red', linestyle='--', label='50% Threshold')
        axes[1].set_title("Missing % per Column")
        axes[1].set_ylabel("% Missing")
        axes[1].legend()
        plt.xticks(rotation=45, ha='right')

        plt.tight_layout()
        plt.show()
        self.log.append("analyze_missing_values: ✅ Done")


    # ------------------------------------------------------------------ #
    #  3. DESCRIPTIVE STATISTICS                                          #
    # ------------------------------------------------------------------ #
    def descriptive_statistics(self):
        """Prints statistical summaries for numerical and categorical columns."""
        print("\n" + "="*60)
        print("  3. DESCRIPTIVE STATISTICS")
        print("="*60)

        if self.num_cols:
            print("\n📊 Numerical Summary:")
            print(self.df[self.num_cols].describe().T.round(3).to_string())

            extra = pd.DataFrame({
                'Skewness':  self.df[self.num_cols].skew().round(3),
                'Kurtosis':  self.df[self.num_cols].kurtosis().round(3),
                'Zeros':     (self.df[self.num_cols] == 0).sum(),
                'Negatives': (self.df[self.num_cols] < 0).sum()
            })
            print("\n📐 Shape Statistics:")
            print(extra.to_string())

            skewed = extra[abs(extra['Skewness']) > 1].index.tolist()
            if skewed:
                print(f"\n⚠️  Highly Skewed Columns (|skew| > 1): {skewed}")

        if self.cat_cols:
            print("\n📊 Categorical Summary:")
            print(self.df[self.cat_cols].describe().to_string())

        self.log.append("descriptive_statistics: ✅ Done")


    # ------------------------------------------------------------------ #
    #  4. UNIVARIATE ANALYSIS                                             #
    # ------------------------------------------------------------------ #
    def univariate_analysis(self, max_cols: int = 8):
        """Plots distributions for all numerical and categorical columns."""
        print("\n" + "="*60)
        print("  4. UNIVARIATE ANALYSIS")
        print("="*60)

        # --- Numerical ---
        num_to_plot = self.num_cols[:max_cols]
        if num_to_plot:
            n = len(num_to_plot)
            fig, axes = plt.subplots(n, 2, figsize=(13, 4 * n))
            if n == 1:
                axes = [axes]

            for i, col in enumerate(num_to_plot):
                data = self.df[col].dropna()

                # Histogram
                axes[i][0].hist(data, bins=30, color='steelblue',
                                 edgecolor='white', alpha=0.85)
                axes[i][0].axvline(data.mean(), color='red',
                                   linestyle='--', label=f'Mean={data.mean():.1f}')
                axes[i][0].axvline(data.median(), color='orange',
                                   linestyle='--', label=f'Median={data.median():.1f}')
                axes[i][0].set_title(f'{col} — Histogram (Skew: {data.skew():.2f})')
                axes[i][0].legend(fontsize=9)

                # KDE
                sns.kdeplot(data, ax=axes[i][1], fill=True, color='steelblue')
                axes[i][1].set_title(f'{col} — Density (KDE)')

            plt.suptitle("Univariate Analysis — Numerical Features", y=1.01, fontsize=13)
            plt.tight_layout()
            plt.show()

        # --- Categorical ---
        cat_to_plot = self.cat_cols[:max_cols]
        if cat_to_plot:
            n = len(cat_to_plot)
            fig, axes = plt.subplots(n, 1, figsize=(10, 4 * n))
            if n == 1:
                axes = [axes]

            for i, col in enumerate(cat_to_plot):
                vc = self.df[col].value_counts()
                colors = sns.color_palette('Set2', len(vc))
                axes[i].bar(vc.index.astype(str), vc.values,
                             color=colors, edgecolor='white')
                axes[i].set_title(f'{col} — Value Counts (Unique: {self.df[col].nunique()})')
                axes[i].set_ylabel("Count")
                axes[i].tick_params(axis='x', rotation=30)

            plt.suptitle("Univariate Analysis — Categorical Features", y=1.01, fontsize=13)
            plt.tight_layout()
            plt.show()

        self.log.append("univariate_analysis: ✅ Done")


    # ------------------------------------------------------------------ #
    #  5. BIVARIATE ANALYSIS                                              #
    # ------------------------------------------------------------------ #
    def bivariate_analysis(self, max_pairs: int = 6):
        """
        Runs Cat vs Num, Num vs Num, and Cat vs Cat analyses
        against the target variable (if provided) and across features.
        """
        print("\n" + "="*60)
        print("  5. BIVARIATE ANALYSIS")
        print("="*60)

        # --- Num vs Num: Pair Plot ---
        if len(self.num_cols) >= 2:
            cols = self.num_cols[:min(max_pairs, len(self.num_cols))]
            hue = self.target if (self.target and self.target_type == 'categorical') else None
            sns.pairplot(self.df[cols + ([self.target] if hue else [])],
                         hue=hue, diag_kind='kde', plot_kws={'alpha': 0.5})
            plt.suptitle("Pair Plot — Numerical Features", y=1.01)
            plt.show()

        # --- Cat vs Num: Target vs Each Categorical ---
        if self.target and self.target_type == 'numerical' and self.cat_cols:
            n = min(4, len(self.cat_cols))
            fig, axes = plt.subplots(1, n, figsize=(5 * n, 5))
            if n == 1:
                axes = [axes]
            for i, col in enumerate(self.cat_cols[:n]):
                sns.boxplot(data=self.df, x=col, y=self.target,
                            palette='Set2', ax=axes[i])
                axes[i].set_title(f'{col} vs {self.target}')
                axes[i].tick_params(axis='x', rotation=30)
            plt.suptitle("Categorical vs Target (Box Plots)")
            plt.tight_layout()
            plt.show()

        # --- Cat vs Cat: Chi-Square between categoricals ---
        if len(self.cat_cols) >= 2:
            print("\n📊 Chi-Square Association Between Categorical Pairs:")
            pairs_checked = 0
            for i in range(len(self.cat_cols)):
                for j in range(i + 1, len(self.cat_cols)):
                    if pairs_checked >= max_pairs:
                        break
                    col1, col2 = self.cat_cols[i], self.cat_cols[j]
                    try:
                        ct = pd.crosstab(self.df[col1], self.df[col2])
                        chi2, p, _, _ = chi2_contingency(ct)
                        sig = "✅ Dependent" if p < 0.05 else "❌ Independent"
                        print(f"  {col1} × {col2}: χ²={chi2:.2f}, p={p:.4f} → {sig}")
                        pairs_checked += 1
                    except Exception:
                        pass

        self.log.append("bivariate_analysis: ✅ Done")


    # ------------------------------------------------------------------ #
    #  6. CORRELATION ANALYSIS                                            #
    # ------------------------------------------------------------------ #
    def correlation_analysis(self, method: str = 'pearson', threshold: float = 0.85):
        """
        Plots a correlation heatmap and identifies highly correlated feature pairs.

        Parameters:
            method:    'pearson', 'spearman', or 'kendall'
            threshold: Correlation above this is flagged as multicollinearity risk
        """
        print("\n" + "="*60)
        print("  6. CORRELATION ANALYSIS")
        print("="*60)

        if len(self.num_cols) < 2:
            print("⚠️  Need at least 2 numerical columns for correlation analysis.")
            return

        corr = self.df[self.num_cols].corr(method=method)

        # --- Heatmap ---
        plt.figure(figsize=(max(8, len(self.num_cols)), max(6, len(self.num_cols) - 2)))
        mask = np.triu(np.ones_like(corr, dtype=bool))
        sns.heatmap(corr, annot=True, fmt='.2f', mask=mask,
                    cmap='coolwarm', center=0, linewidths=0.5,
                    vmin=-1, vmax=1,
                    annot_kws={'size': max(6, 10 - len(self.num_cols) // 3)})
        plt.title(f"Correlation Heatmap ({method.title()})")
        plt.tight_layout()
        plt.show()

        # --- High correlation pairs ---
        high_corr = []
        for i in range(len(corr.columns)):
            for j in range(i):
                val = abs(corr.iloc[i, j])
                if val >= threshold:
                    high_corr.append({
                        'Feature 1':   corr.columns[i],
                        'Feature 2':   corr.columns[j],
                        'Correlation': round(corr.iloc[i, j], 3)
                    })

        if high_corr:
            print(f"\n⚠️  Highly Correlated Pairs (|r| ≥ {threshold}):")
            print(pd.DataFrame(high_corr).to_string(index=False))
            print("   → Consider dropping one from each pair to reduce multicollinearity.")
        else:
            print(f"✅ No feature pairs with |r| ≥ {threshold} found.")

        # --- Target correlations ---
        if self.target and self.target in self.num_cols:
            tc = corr[self.target].drop(self.target).sort_values(key=abs, ascending=False)
            plt.figure(figsize=(8, max(4, len(tc) // 2)))
            colors = ['seagreen' if v > 0 else 'tomato' for v in tc]
            tc.plot(kind='barh', color=colors)
            plt.axvline(0, color='black', linewidth=0.8)
            plt.title(f"Feature Correlation with Target: {self.target}")
            plt.xlabel(f'{method.title()} r')
            plt.tight_layout()
            plt.show()

        self.log.append(f"correlation_analysis: ✅ Done (method={method})")


    # ------------------------------------------------------------------ #
    #  7. OUTLIER ANALYSIS                                                #
    # ------------------------------------------------------------------ #
    def outlier_analysis(self):
        """Visualizes and counts outliers using IQR and Z-Score methods."""
        print("\n" + "="*60)
        print("  7. OUTLIER ANALYSIS")
        print("="*60)

        if not self.num_cols:
            print("⚠️  No numerical columns to check.")
            return

        # --- Box Plots ---
        n = len(self.num_cols)
        ncols = min(3, n)
        nrows = (n + ncols - 1) // ncols
        fig, axes = plt.subplots(nrows, ncols, figsize=(5 * ncols, 4 * nrows))
        axes = np.array(axes).flatten()

        for i, col in enumerate(self.num_cols):
            axes[i].boxplot(self.df[col].dropna(), vert=True, patch_artist=True,
                            boxprops=dict(facecolor='steelblue', alpha=0.6),
                            medianprops=dict(color='red', linewidth=2))
            axes[i].set_title(col)

        for j in range(i + 1, len(axes)):
            axes[j].set_visible(False)

        plt.suptitle("Box Plots — Outlier Overview", fontsize=13)
        plt.tight_layout()
        plt.show()

        # --- Outlier Count Table ---
        records = []
        for col in self.num_cols:
            Q1, Q3 = self.df[col].quantile(0.25), self.df[col].quantile(0.75)
            IQR = Q3 - Q1
            n_iqr = ((self.df[col] < Q1 - 1.5*IQR) |
                     (self.df[col] > Q3 + 1.5*IQR)).sum()

            zs = np.abs(stats.zscore(self.df[col].dropna()))
            n_z = (zs > 3).sum()

            records.append({
                'Column':       col,
                'IQR Outliers': n_iqr,
                'IQR %':        round(n_iqr / len(self.df) * 100, 2),
                'Z Outliers':   n_z,
                'Z %':          round(n_z / len(self.df) * 100, 2)
            })

        result = pd.DataFrame(records).set_index('Column')
        print("\n📊 Outlier Summary:")
        print(result.to_string())

        self.log.append("outlier_analysis: ✅ Done")


    # ------------------------------------------------------------------ #
    #  8. TARGET VARIABLE ANALYSIS                                        #
    # ------------------------------------------------------------------ #
    def analyze_target(self):
        """Analyzes and visualizes the target variable in depth."""
        if not self.target:
            print("⚠️  No target column specified.")
            return

        print("\n" + "="*60)
        print(f"  8. TARGET VARIABLE ANALYSIS: '{self.target}'")
        print("="*60)

        target_data = self.df[self.target]

        if self.target_type == 'numerical':
            fig, axes = plt.subplots(1, 3, figsize=(16, 4))

            axes[0].hist(target_data.dropna(), bins=30,
                         color='steelblue', edgecolor='white')
            axes[0].axvline(target_data.mean(), color='red', linestyle='--',
                            label=f'Mean={target_data.mean():.1f}')
            axes[0].axvline(target_data.median(), color='orange', linestyle='--',
                            label=f'Median={target_data.median():.1f}')
            axes[0].set_title(f'{self.target} Distribution')
            axes[0].legend()

            axes[1].boxplot(target_data.dropna(), patch_artist=True,
                            boxprops=dict(facecolor='steelblue', alpha=0.6))
            axes[1].set_title(f'{self.target} Box Plot')

            stats.probplot(target_data.dropna(), dist='norm', plot=axes[2])
            axes[2].set_title(f'{self.target} QQ Plot')

            plt.suptitle(f"Target Variable: {self.target}")
            plt.tight_layout()
            plt.show()

            print(f"\n{target_data.describe().round(2).to_string()}")
            print(f"Skewness: {target_data.skew():.3f}")
            if abs(target_data.skew()) > 1:
                print("⚠️  Target is skewed — consider log-transforming the target.")

        elif self.target_type == 'categorical':
            vc = target_data.value_counts()
            pct = target_data.value_counts(normalize=True) * 100

            print("\nClass Distribution:")
            summary = pd.concat([vc, pct.round(2)],
                                 axis=1, keys=['Count', 'Percent'])
            print(summary.to_string())

            fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
            colors = sns.color_palette('Set2', len(vc))

            ax1.bar(vc.index.astype(str), vc.values,
                    color=colors, edgecolor='white')
            ax1.set_title(f'{self.target} — Class Counts')
            ax1.set_ylabel('Count')
            ax1.tick_params(axis='x', rotation=30)

            ax2.pie(pct.values, labels=pct.index.astype(str),
                    autopct='%1.1f%%', colors=colors, startangle=90)
            ax2.set_title(f'{self.target} — Proportions')

            plt.suptitle(f"Target: {self.target}")
            plt.tight_layout()
            plt.show()

            ratio = vc.max() / vc.min()
            print(f"\nImbalance Ratio: {ratio:.1f}:1")
            if ratio > 5:
                print("⚠️  Severe imbalance! Use SMOTE, class_weight, or resampling.")
            elif ratio > 2:
                print("⚠️  Moderate imbalance — consider oversampling minority class.")
            else:
                print("✅ Classes are roughly balanced.")

        self.log.append("analyze_target: ✅ Done")


    # ------------------------------------------------------------------ #
    #  9. DISTRIBUTION & SKEWNESS                                         #
    # ------------------------------------------------------------------ #
    def distribution_analysis(self):
        """QQ Plots and skewness flags for all numerical columns."""
        print("\n" + "="*60)
        print("  9. DISTRIBUTION & SKEWNESS ANALYSIS")
        print("="*60)

        if not self.num_cols:
            print("⚠️  No numerical columns found.")
            return

        n = len(self.num_cols)
        fig, axes = plt.subplots(n, 1, figsize=(6, 4 * n))
        if n == 1:
            axes = [axes]

        for i, col in enumerate(self.num_cols):
            stats.probplot(self.df[col].dropna(), dist='norm', plot=axes[i])
            skew_val = self.df[col].skew()
            axes[i].set_title(f'{col} — QQ Plot (Skew: {skew_val:.2f})')

        plt.suptitle("QQ Plots — Normality Check", y=1.01, fontsize=13)
        plt.tight_layout()
        plt.show()

        self.log.append("distribution_analysis: ✅ Done")


    # ------------------------------------------------------------------ #
    #  10. FEATURE IMPORTANCE                                             #
    # ------------------------------------------------------------------ #
    def feature_importance(self):
        """Quick Random Forest feature importance plot."""
        if not self.target:
            print("⚠️  No target column specified for feature importance.")
            return

        print("\n" + "="*60)
        print("  10. FEATURE IMPORTANCE (Random Forest)")
        print("="*60)

        try:
            from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
            from sklearn.preprocessing import LabelEncoder

            df_clean = self.df.dropna()
            X = df_clean.drop(columns=[self.target])
            y = df_clean[self.target]

            for col in X.select_dtypes(include='object').columns:
                X[col] = LabelEncoder().fit_transform(X[col].astype(str))

            if self.target_type == 'categorical':
                if y.dtype == 'object':
                    y = LabelEncoder().fit_transform(y.astype(str))
                model = RandomForestClassifier(n_estimators=100, random_state=42)
            else:
                model = RandomForestRegressor(n_estimators=100, random_state=42)

            model.fit(X, y)

            importance = pd.Series(model.feature_importances_,
                                   index=X.columns).sort_values(ascending=True)

            plt.figure(figsize=(9, max(5, len(importance) * 0.4)))
            importance.plot(kind='barh', color='steelblue', edgecolor='white')
            plt.axvline(importance.mean(), color='red', linestyle='--',
                        label='Mean Importance')
            plt.title(f"Feature Importance for Target: {self.target}")
            plt.xlabel("Importance Score")
            plt.legend()
            plt.tight_layout()
            plt.show()

            low_imp = importance[importance < importance.mean() * 0.1].index.tolist()
            if low_imp:
                print(f"\n⚠️  Very Low Importance Features (consider dropping):\n  {low_imp}")

            self.log.append("feature_importance: ✅ Done")

        except ImportError:
            print("⚠️  scikit-learn not installed. Run: pip install scikit-learn")


    # ------------------------------------------------------------------ #
    #  SUMMARY                                                            #
    # ------------------------------------------------------------------ #
    def print_summary(self):
        """Prints a structured summary of all EDA steps completed."""
        print("\n" + "="*60)
        print("       EDA MANAGER — COMPLETION SUMMARY")
        print("="*60)
        print(f"  Dataset: {self.n_rows} rows × {self.n_cols} columns")
        print(f"  Numerical Columns  : {len(self.num_cols)}")
        print(f"  Categorical Columns: {len(self.cat_cols)}")
        if self.target:
            print(f"  Target Column      : '{self.target}' ({self.target_type})")
        print(f"\n  Steps Completed: {len(self.log)}")
        print("-"*60)
        for i, entry in enumerate(self.log, 1):
            print(f"  {i:2d}. {entry}")
        print("="*60)


    # ------------------------------------------------------------------ #
    #  FULL PIPELINE                                                      #
    # ------------------------------------------------------------------ #
    def run_full_eda(self, max_cols: int = 8):
        """
        Runs the complete EDA pipeline in the recommended sequence.

        Parameters:
            max_cols: Max columns to plot in univariate/bivariate charts.
        """
        print("\n🚀 Starting Full EDA Pipeline...\n")
        self.inspect_shape()
        self.analyze_missing_values()
        self.descriptive_statistics()
        self.univariate_analysis(max_cols=max_cols)
        self.bivariate_analysis(max_pairs=max_cols)
        self.correlation_analysis()
        self.outlier_analysis()
        self.distribution_analysis()
        if self.target:
            self.analyze_target()
            self.feature_importance()
        self.print_summary()
        print("\n✅ Full EDA Complete!")


# ============================================================ #
#  EXAMPLE USAGE
# ============================================================ #
if __name__ == "__main__":
    import numpy as np

    # Simulated raw dataset
    np.random.seed(42)
    n = 500
    raw_df = pd.DataFrame({
        'Age':         np.random.randint(18, 70, n).astype(float),
        'Salary':      np.random.exponential(50000, n),   # Right-skewed
        'YearsExp':    np.random.randint(0, 30, n).astype(float),
        'Department':  np.random.choice(['IT', 'HR', 'Finance', 'Sales'], n),
        'Gender':      np.random.choice(['Male', 'Female', 'Other'], n,
                                         p=[0.5, 0.45, 0.05]),
        'Promoted':    np.random.choice([0, 1], n, p=[0.75, 0.25]),  # Imbalanced target
    })

    # Inject missing values
    raw_df.loc[np.random.choice(n, 40, replace=False), 'Age']     = np.nan
    raw_df.loc[np.random.choice(n, 20, replace=False), 'Salary']  = np.nan
    raw_df.loc[np.random.choice(n, 15, replace=False), 'Gender']  = np.nan

    # Inject outliers
    raw_df.loc[0, 'Salary'] = 2_000_000
    raw_df.loc[1, 'Age']    = 150

    print("Raw Data Sample:")
    print(raw_df.head())

    # --- Run EDA BEFORE Preprocessing ---
    print("\n" + "🔍 EDA BEFORE PREPROCESSING ".center(60, "="))
    eda_before = EDAManager(raw_df, target='Promoted', target_type='categorical')
    eda_before.run_full_eda()

    # --- Minimal Preprocessing ---
    from sklearn.impute import SimpleImputer
    from sklearn.preprocessing import LabelEncoder

    df_proc = raw_df.copy()
    df_proc['Age'].fillna(df_proc['Age'].median(), inplace=True)
    df_proc['Salary'].fillna(df_proc['Salary'].median(), inplace=True)
    df_proc['Gender'].fillna('Unknown', inplace=True)
    df_proc['Salary'] = np.log1p(df_proc['Salary'])
    df_proc['Department'] = LabelEncoder().fit_transform(df_proc['Department'])
    df_proc['Gender']     = LabelEncoder().fit_transform(df_proc['Gender'])

    # --- Run EDA AFTER Preprocessing ---
    print("\n" + "✅ EDA AFTER PREPROCESSING ".center(60, "="))
    eda_after = EDAManager(df_proc, target='Promoted', target_type='categorical')
    eda_after.run_full_eda()
```

---

## 📊 EDA Workflow Cheat Sheet

```
Raw Dataset
    │
    ▼
1. Inspect Shape & Types        → What am I working with?
    │
    ▼
2. Missing Value Analysis       → Where is data absent?
    │
    ▼
3. Descriptive Statistics       → What are typical values? Any anomalies?
    │
    ▼
4. Univariate Analysis          → How is each feature distributed?
    │
    ▼
5. Bivariate Analysis           → How do features relate to each other?
    │
    ▼
6. Correlation Analysis         → Which features are linearly associated?
    │
    ▼
7. Outlier Analysis             → What extreme values exist?
    │
    ▼
8. Distribution / Skewness      → Which columns need transformation?
    │
    ▼
9. Target Variable Analysis     → Is target skewed? Imbalanced?
    │
    ▼
10. Feature Importance          → Which features actually matter?
    │
    ▼
  → Preprocessing Plan (before)
  → Modeling Plan (after)
```

---

> **💡 Golden Rule of EDA:** Never skip it. 80% of data science work is understanding and cleaning data. A model trained on poorly-understood data is just an expensive way to amplify your assumptions.

> **🔁 Before & After Rule:** Always re-run EDA after preprocessing. Confirm that distributions improved, missing values are gone, outliers are handled, and classes are balanced — *before* you ever touch a model.
