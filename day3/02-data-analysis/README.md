# Data Analysis, Validation and Leakage

Model-এর performance অনেক সময় algorithm-এর চেয়ে data quality ও data collection process-এর উপর বেশি নির্ভর করে। EDA (Exploratory Data Analysis) হলো model training-এর আগে data structure, distribution, missingness, outlier, relationship এবং anomaly বোঝার কাজ।

## ১. Exploratory Data Analysis কীভাবে শুরু করব?

~~~python
import pandas as pd

df = pd.read_csv("transactions.csv")

print(df.shape)
print(df.info())
print(df.describe(include="all"))
print(df.isna().sum().sort_values(ascending=False))
print(df["is_fraud"].value_counts(normalize=True))
print(df.nunique())
~~~

প্রাথমিক EDA checklist:
- Row/column count ও duplicate।
- Column dtype ও invalid value।
- Missing count ও missing percentage।
- Numeric distribution ও outlier।
- Categorical unique value ও rare category।
- Target distribution ও feature-target relationship।
- ID, timestamp এবং potential leakage column।

~~~mermaid
flowchart TD
    A["Raw dataset"] --> B["Schema and types"]
    B --> C["Missing / duplicates"]
    C --> D["Distribution / outliers"]
    D --> E["Target and feature relationship"]
    E --> F["Leakage and split strategy"]
    F --> G["Document data quality findings"]
~~~

## ২. Missing Data কী? MCAR, MAR এবং MNAR কী?

Missing data-এর pattern বুঝতে পারলে drop বা imputation decision ভালো হয়।

- **MCAR (Missing Completely At Random):** missing হওয়া observed/unobserved data-এর সঙ্গে সম্পর্কিত নয়।
- **MAR (Missing At Random):** missing হওয়া observed variable-এর উপর নির্ভর করে।
- **MNAR (Missing Not At Random):** missing হওয়ার কারণ missing value নিজেই বা unobserved factor-এর সঙ্গে সম্পর্কিত।

~~~mermaid
flowchart TD
    A["Why is a value missing?"] --> B["No systematic relationship"]
    A --> C["Depends on observed variables"]
    A --> D["Depends on missing value / unobserved factor"]
    B --> E["MCAR"]
    C --> F["MAR"]
    D --> G["MNAR"]
~~~

~~~python
missing_rate = df.isna().mean().sort_values(ascending=False)
print(missing_rate)
~~~

Drop, imputation, missing indicator বা explicit missing category বেছে নেওয়া যায়। Missing value zero দিয়ে fill করার আগে zero-এর domain meaning বুঝতে হবে।

## ৩. Missing value imputation কীভাবে বেছে নেব?

- Numeric skewed data: median বিবেচনা করা যায়।
- Numeric symmetric data: mean উপযোগী হতে পারে।
- Categorical: most frequent বা explicit "unknown" category।
- Missingness informative হলে missing indicator useful হতে পারে।

কোনো strategy universal নয়। Imputer শুধু training data-তে fit করতে হবে, validation/test data-তে transform।

## ৪. Outlier কী? সব outlier remove করা উচিত?

Outlier data error, rare legitimate event অথবা fraud signal হতে পারে। অন্ধভাবে remove করবে না।

~~~mermaid
flowchart TD
    A["Unusual observation"] --> B{"Data entry / sensor error?"}
    B -->|হ্যাঁ| C["Correct or mark invalid"]
    B -->|না| D{"Valid but rare event?"}
    D -->|হ্যাঁ| E["Keep; consider robust methods"]
    D -->|অজানা| F["Investigate domain context"]
~~~

Boxplot, quantile, IQR, robust statistics ও domain rule দিয়ে investigate করো। Outlier handling rule train/test boundary এবং business policy মেনে করতে হবে।

## ৫. Correlation কী এবং কেন causation নয়?

Correlation দুই variable-এর association measure করে। Correlation থাকলেই একটি variable অন্যটির cause প্রমাণ হয় না। Confounder, reverse causality এবং selection bias association তৈরি করতে পারে।

~~~mermaid
flowchart TD
    A["Observed correlation"] --> B["Possible direct relationship"]
    A --> C["Confounding variable"]
    A --> D["Reverse causality"]
    A --> E["Selection / measurement bias"]
    B --> F["Need causal design / domain evidence"]
    C --> F
    D --> F
    E --> F
~~~

Correlation matrix EDA-তে useful, কিন্তু feature importance বা correlation থেকে causal conclusion টানা যাবে না।

## ৬. Distribution Shift কী?

Training এবং production data-র distribution বদলে গেলে model performance কমতে পারে।

- **Data drift:** input feature distribution বদলায়।
- **Label shift:** target class prior/distribution বদলায়।
- **Concept drift:** input ও target-এর relationship বদলায়।

~~~mermaid
flowchart TD
    A["Production behavior changes"] --> B["Feature distribution changes"]
    A --> C["Target prevalence changes"]
    A --> D["Feature-target relationship changes"]
    B --> E["Data drift"]
    C --> F["Label shift"]
    D --> G["Concept drift"]
~~~

Drift detect করলেই retrain করতে হবে এমন নয়। প্রথমে measurement pipeline, seasonality, product changes এবং label delay investigate করো।

## ৭. Data Leakage-এর ধরন কী?

Leakage হলো training/evaluation-এ এমন তথ্য ঢুকে যাওয়া যা বাস্তব prediction time-এ পাওয়া সম্ভব নয়, অথবা test data-র information training process-এ প্রভাব ফেলা।

- **Target leakage:** target-এর পরের তথ্য feature-এ ঢুকে যাওয়া।
- **Train-test contamination:** test data preprocessing/training-এ প্রভাব ফেলা।
- **Temporal leakage:** future observation দিয়ে past predict করা।
- **Group leakage:** একই user/patient/entity-এর correlated record train ও test-এ ছড়িয়ে পড়া।

~~~mermaid
flowchart TD
    A["Potential leakage"] --> B["Future / post-outcome feature"]
    A --> C["Fit preprocessing before split"]
    A --> D["Same entity across train and test"]
    A --> E["Future rows used to predict past"]
    B --> F["Unrealistically high evaluation"]
    C --> F
    D --> F
    E --> F
~~~

## ৮. Time-based Split কখন দরকার?

যদি ভবিষ্যৎ transaction predict করতে হয়, পুরোনো সময় training এবং পরের সময় validation/test রাখো। Random split বাস্তব deployment condition নাও প্রতিফলিত করতে পারে।

~~~mermaid
flowchart LR
    A["Past"] --> B["Train"]
    B --> C["Validation"]
    C --> D["Future test"]
    D --> E["Simulated deployment"]
~~~

Temporal feature তৈরি করার সময় look-ahead window, rolling statistic এবং target availability timestamp পরীক্ষা করো।

## ৯. Group-based Split কখন দরকার?

একই customer, patient, device বা household-এর multiple record থাকলে random split একই entity-র তথ্য train ও test উভয় জায়গায় রাখতে পারে। এতে model entity-specific pattern memorize করে misleading score দিতে পারে।

Group-aware split-এ group পুরোপুরি একটি split-এ রাখা হয়। Deployment scenario যদি unseen customer হয়, group split বেশি realistic।

## ১০. Feature Importance কী? এটি কি causality প্রমাণ করে?

Permutation importance feature shuffle করে score কত কমে তা মাপে। Correlated feature থাকলে interpretation জটিল হতে পারে। Feature importance prediction behavior সম্পর্কে insight দেয়, causality প্রমাণ করে না।

SHAP-এর মতো method individual prediction explain করতে সাহায্য করে, কিন্তু explanation method-এর assumption ও limitation আছে।

## ১১. SQL বনাম Pandas কখন ব্যবহার করব?

SQL database-এ filtering/join/aggregation pushdown করতে পারে; Pandas in-memory analysis ও Python workflow-এর জন্য সুবিধাজনক। বড় dataset হলে data database-এই aggregate করা efficient হতে পারে।

| SQL | Pandas |
|---|---|
| Database engine-এ query execute | Python process memory-তে DataFrame |
| Large table filter/join/aggregate pushdown | Exploratory analysis ও custom Python transform |
| Index/query planner leverage করতে পারে | Vectorized in-memory operations |
| Network transfer কমাতে পারে | RAM সীমা বিবেচনা করতে হয় |

## ১২. Data Quality Report-এ কী রাখব?

- Dataset source ও extraction time।
- Row/column count।
- Schema ও dtype।
- Missing percentage।
- Duplicate key count।
- Invalid value count।
- Target class distribution।
- Outlier investigation summary।
- Time/group dependency।
- Leakage risk এবং remediation।
- Accepted/rejected row count reconciliation।

## Practice

একটি dataset-এ:
1. Missingness, duplicates, outliers, class balance ও temporal pattern বিশ্লেষণ করো।
2. MCAR/MAR/MNAR-এর সম্ভাব্য ব্যাখ্যা লেখো।
3. Leakage-এর অন্তত তিনটি সম্ভাব্য উদাহরণ চিহ্নিত করো।
4. Random, time-based এবং group-based split-এর উপযুক্ততা বিচার করো।
5. Feature correlation ও importance report করো, কিন্তু causal claim কোরো না।
6. একটি concise data quality report তৈরি করো।

## Practice করার পদ্ধতি

1. Cleaning করার আগে raw data snapshot ও row count record করো।
2. Missing/outlier handling-এর business reason লিখে রাখো।
3. Preprocessing rule training data-তে fit করো।
4. Prediction time-এ কোন feature available তা verify করো।
5. Accepted ও rejected record count reconcile করো।
