# Machine Learning Algorithms

এই নোটে algorithm-এর intuition, typical use case ও limitation তুলে ধরা হয়েছে।

## ১. Linear Regression

Continuous target predict করার জন্য input feature-এর weighted linear combination শেখে।

```text
prediction = w1*x1 + w2*x2 + ... + b
```

ব্যবহার: baseline regression, trend estimation, interpretable relationship।

Limitation: linear relationship assumption; outlier ও multicollinearity প্রভাব ফেলতে পারে।

## ২. Logistic Regression

নামের মধ্যে Regression থাকলেও এটি classification algorithm। Linear combination-কে sigmoid function দিয়ে probability-তে map করে।

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)
probabilities = model.predict_proba(X_test)
predictions = model.predict(X_test)
```

Binary classification-এ বহুল ব্যবহৃত baseline। Feature scaling উপকারী হতে পারে।

## ৩. Decision Tree

Feature-এর threshold ধরে data-কে ধাপে ধাপে split করে prediction দেয়। সহজে ব্যাখ্যা করা যায়, numerical scaling সাধারণত দরকার হয় না।

Limitation: গভীর tree overfit করতে পারে এবং data-র ছোট পরিবর্তনে structure বদলাতে পারে।

## ৪. Random Forest

অনেক Decision Tree train করে তাদের prediction combine করে। Classification-এ voting এবং regression-এ averaging ব্যবহৃত হয়।

- Bagging ও random feature selection variance কমাতে সাহায্য করে।
- Nonlinear relationship ধরতে পারে।
- Interpretability একক tree-এর তুলনায় কম।
- Model বড় হতে পারে।

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=200,
    max_depth=None,
    random_state=42,
    class_weight="balanced",
)
```

## ৫. K-Nearest Neighbors (KNN)

নতুন point-এর কাছের Kটি training sample দেখে prediction করে। Distance metric গুরুত্বপূর্ণ; feature scaling সাধারণত প্রয়োজন।

Limitation: বড় dataset-এ inference ধীর হতে পারে; high-dimensional space-এ distance কম informative হতে পারে।

## ৬. K-Means Clustering

Unlabeled data-কে Kটি cluster-এ ভাগ করার চেষ্টা করে, cluster centroid-এর distance minimize করে।

```python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, random_state=42, n_init="auto")
cluster_ids = model.fit_predict(X_scaled)
```

K আগে নির্ধারণ করতে হয়। Feature scaling গুরুত্বপূর্ণ; outlier ও non-spherical cluster-এ সমস্যা হতে পারে।

## ৭. XGBoost / Gradient Boosting

Boosting sequentially weak learners (সাধারণত shallow trees) যোগ করে আগের model-এর error কমানোর চেষ্টা করে। Tabular data-তে শক্তিশালী হতে পারে।

Trade-off:
- Hyperparameter tuning দরকার।
- Overfit করতে পারে।
- Training cost বেশি হতে পারে।
- Feature importance সতর্কভাবে interpret করতে হয়।

## ৮. কোন algorithm কখন?

| পরিস্থিতি | সম্ভাব্য starting point |
|---|---|
| Simple interpretable binary classification | Logistic Regression |
| Nonlinear tabular data | Random Forest |
| Strong tabular performance target | Gradient Boosting / XGBoost |
| ছোট dataset, local similarity meaningful | KNN |
| Unlabeled segmentation | K-Means |
| Continuous numeric target | Linear Regression / tree-based regressor |

কোনো algorithm সর্বক্ষেত্রে সেরা নয়। Baseline, validation এবং domain knowledge দিয়ে সিদ্ধান্ত নিতে হবে।

## ৯. Feature Scaling

অনেক algorithm distance বা gradient-এর উপর নির্ভর করে, তাই feature scale গুরুত্বপূর্ণ।

- StandardScaler: mean 0, standard deviation 1 করার চেষ্টা।
- MinMaxScaler: নির্দিষ্ট range-এ scale।
- Tree-based model সাধারণত scaling-এর উপর কম নির্ভরশীল।

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

গুরুত্বপূর্ণ: scaler শুধু training data-তে fit করবে। Test data-তে শুধু transform।

## ১০. Regularization

Model-এর complexity নিয়ন্ত্রণ করতে loss function-এ penalty যোগ করা হয়।

- L1 (Lasso): কিছু coefficient zero করতে পারে।
- L2 (Ridge): coefficient ছোট করে, সাধারণত zero করে না।
- Logistic Regression-এও regularization ব্যবহৃত হয়।

## Interview Practice

1. Logistic Regression কেন classification algorithm?
2. Decision Tree overfit করে কেন?
3. Random Forest কীভাবে variance কমায়?
4. KNN-এ scaling কেন গুরুত্বপূর্ণ?
5. K-Means-এর limitation কী?
6. Boosting ও bagging-এর high-level পার্থক্য কী?
7. Linear model-এর regularization কেন দরকার?
