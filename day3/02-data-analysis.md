# Data Analysis, Validation and Leakage

## ১. Exploratory Data Analysis (EDA)
EDA হলো model training-এর আগে data structure, distribution, missingness, outlier, relationship ও anomaly বোঝার প্রক্রিয়া।

```python
import pandas as pd

df = pd.read_csv("transactions.csv")
print(df.shape)
print(df.info())
print(df.describe(include="all"))
print(df.isna().sum().sort_values(ascending=False))
print(df["is_fraud"].value_counts(normalize=True))
```

## ২. Missing Data
Missing value MCAR, MAR বা MNAR ধরনের হতে পারে। বাস্তবে কারণ জানা কঠিন, তাই missingness pattern ও domain context পরীক্ষা করতে হবে। Drop, imputation বা missing indicator বেছে নেওয়া যায়।

## ৩. Outlier
Outlier data error, rare legitimate event অথবা fraud signal হতে পারে। অন্ধভাবে remove করবে না। Boxplot, quantile, IQR ও domain rule দিয়ে investigate করো।

## ৪. Distribution Shift
Training এবং production data-র feature/label distribution বদলে গেলে model performance কমতে পারে। Data drift ও concept drift আলাদা করে বোঝা দরকার:
- Data drift: input distribution বদলায়।
- Concept drift: input ও target-এর relationship বদলায়।

## ৫. Data Leakage-এর ধরন
- Target leakage: target-এর পরের তথ্য feature-এ ঢুকে যাওয়া।
- Train-test contamination: test data preprocessing/training-এ প্রভাব ফেলা।
- Temporal leakage: future observation দিয়ে past predict করা।

## ৬. Time-based Split
যদি ভবিষ্যৎ transaction predict করতে হয়, পুরোনো সময় training এবং পরের সময় test রাখো। Random split বাস্তব deployment condition নাও প্রতিফলিত করতে পারে।

## ৭. Feature Importance
Permutation importance feature shuffle করে score কত কমে তা মাপে। Correlated feature থাকলে interpretation জটিল হতে পারে। Feature importance causality প্রমাণ করে না।

## ৮. SQL বনাম Pandas
SQL database-এ filtering/join/aggregation pushdown করতে পারে; Pandas in-memory analysis-এর জন্য সুবিধাজনক। বড় dataset হলে data database-এই aggregate করা efficient হতে পারে।

## Practice
একটি dataset-এ missingness, duplicates, outliers, class balance ও temporal pattern বিশ্লেষণ করো। Leakage-এর অন্তত তিনটি সম্ভাব্য উদাহরণ লিখে তার prevention দেখাও।
