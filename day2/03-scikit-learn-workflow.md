# Scikit-learn End-to-End Workflow

## ১. Dataset ও Problem

প্রথমে নির্ধারণ করতে হবে:
- কী predict করব?
- Target column কোনটি?
- Prediction-এর সময় কোন feature পাওয়া যাবে?
- ভুল positive ও ভুল negative-এর business impact কী?

## ২. Data Split

```python
import pandas as pd
from sklearn.model_selection import train_test_split

df = pd.read_csv("transactions.csv")
X = df.drop(columns=["is_fraud"])
y = df["is_fraud"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)
```

Real-world data-তে সময়ের ক্রম গুরুত্বপূর্ণ হলে random split-এর বদলে time-based split প্রয়োজন হতে পারে।

## ৩. Preprocessing Pipeline

```python
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.linear_model import LogisticRegression

numeric_features = ["amount", "account_age_days"]
categorical_features = ["merchant_category", "country"]

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
])

categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("onehot", OneHotEncoder(handle_unknown="ignore")),
])

preprocessor = ColumnTransformer([
    ("numeric", numeric_pipeline, numeric_features),
    ("categorical", categorical_pipeline, categorical_features),
])

model = Pipeline([
    ("preprocess", preprocessor),
    ("classifier", LogisticRegression(max_iter=1000, class_weight="balanced")),
])

model.fit(X_train, y_train)
```

Pipeline preprocessing-কে model training-এর সঙ্গে বেঁধে রাখে এবং leakage risk কমাতে সাহায্য করে।

## ৪. Prediction ও Evaluation

```python
from sklearn.metrics import (
    classification_report,
    confusion_matrix,
    roc_auc_score,
)

predictions = model.predict(X_test)
probabilities = model.predict_proba(X_test)[:, 1]

print(confusion_matrix(y_test, predictions))
print(classification_report(y_test, predictions))
print("ROC-AUC:", roc_auc_score(y_test, probabilities))
```

শুধু accuracy দেখে সিদ্ধান্ত নেবে না, বিশেষ করে class imbalance থাকলে।

## ৫. Cross Validation

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X_train, y_train, cv=cv, scoring="f1")
print(scores.mean(), scores.std())
```

Cross-validation training data-র উপর model estimate করতে সাহায্য করে। Final test set untouched রাখবে।

## ৬. Hyperparameter Tuning

```python
from sklearn.model_selection import GridSearchCV

search = GridSearchCV(
    model,
    param_grid={
        "classifier__C": [0.01, 0.1, 1, 10],
    },
    scoring="f1",
    cv=5,
    n_jobs=-1,
)
search.fit(X_train, y_train)

best_model = search.best_estimator_
print(search.best_params_)
```

বড় search space-এ RandomizedSearchCV ব্যবহার করা যেতে পারে।

## ৭. Model Persistence

```python
import joblib

joblib.dump(best_model, "fraud_model.joblib")
loaded_model = joblib.load("fraud_model.joblib")
```

শুধু trusted artifact load করবে। Pickle/joblib untrusted source থেকে load করা নিরাপদ নয়। Production-এ model artifact version, dependencies ও preprocessing compatibility track করতে হবে।

## ৮. Reproducibility

- Random seed নির্দিষ্ট করা।
- Dataset version track করা।
- Feature definition document করা।
- Library version pin করা।
- Training configuration সংরক্ষণ করা।
- Evaluation metric ও split strategy লিখে রাখা।

## ৯. Common Mistakes

- Split করার আগে পুরো dataset preprocess করা।
- Test set দিয়ে hyperparameter tune করা।
- Imbalanced data-তে শুধু accuracy ব্যবহার।
- Training-serving feature mismatch।
- Model probability-কে calibrated probability ধরে নেওয়া।
- Model deploy করে monitoring না করা।

## Practice

নিজের dataset দিয়ে:
1. Problem ও target define করো।
2. Data inspect ও split করো।
3. Numeric/categorical preprocessing pipeline তৈরি করো।
4. Logistic Regression train করো।
5. Confusion matrix, precision, recall, F1 বের করো।
6. Cross-validation চালাও।
7. Model save ও reload করে একই prediction আসে কি না দেখো।
