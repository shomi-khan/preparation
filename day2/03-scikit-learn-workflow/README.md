# Scikit-learn End-to-End Workflow

একটি ML model train করা workflow-এর একটি ধাপ মাত্র। Real project-এ problem framing, data split, preprocessing, baseline, validation, tuning, final test, artifact save এবং monitoring সবকিছু গুরুত্বপূর্ণ। Scikit-learn-এর Pipeline ও ColumnTransformer এই workflow-কে repeatable এবং leakage-resistant করতে সাহায্য করে।

এই chapter-এ classification example ব্যবহার করা হয়েছে। Code-এ ধরে নেওয়া হয়েছে dataset-এর target column হলো is_fraud এবং feature column-গুলো আগে থেকে চিহ্নিত।

## ১. Dataset ও Problem কীভাবে define করব?

প্রথমে নির্ধারণ করো:
- কী predict করব?
- Target column কোনটি?
- Prediction-এর সময় কোন feature বাস্তবে available?
- Prediction horizon কী?
- False positive ও false negative-এর business impact কী?
- Success metric কী হবে?

Fraud detection-এ false negative মানে fraud miss করা; false positive মানে legitimate transaction block করা। কোনটি বেশি costly, তা threshold ও metric choice-এ প্রভাব ফেলবে।

~~~mermaid
flowchart TD
    A["Business problem"] --> B["Define prediction target"]
    B --> C["Define prediction time"]
    C --> D["Identify available features"]
    D --> E["Define error cost and success metric"]
    E --> F["Build ML dataset"]
~~~

## ২. Data inspect এবং leakage check কীভাবে করব?

~~~python
import pandas as pd

df = pd.read_csv("transactions.csv")
print(df.shape)
print(df.dtypes)
print(df.isna().sum())
print(df["is_fraud"].value_counts(normalize=True))
print(df.head())
~~~

Feature হিসেবে এমন column ব্যবহার কোরো না যা prediction-এর পরে তৈরি হয়। ID, timestamp, status update, investigation result ইত্যাদিতে leakage থাকতে পারে। Data split strategy-ও record/group/time dependency অনুযায়ী হওয়া দরকার।

## ৩. Train/Test Split কীভাবে করব?

~~~python
from sklearn.model_selection import train_test_split

X = df.drop(columns=["is_fraud"])
y = df["is_fraud"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)
~~~

- `test_size=0.2`: ২০% test data।
- `random_state`: reproducible split।
- `stratify=y`: classification-এ class proportion কাছাকাছি রাখে।

~~~mermaid
flowchart TD
    A["Full labeled dataset"] --> B["Training data"]
    A --> C["Hold-out test data"]
    B --> D["Fit / CV / tune"]
    D --> E["Freeze model and decisions"]
    E --> F["Evaluate once on test"]
~~~

**Split strategy problem অনুযায়ী বেছে নাও:**
- Independent IID sample: random/stratified split।
- একই user/patient/group-এর multiple record: group-aware split।
- Time-dependent prediction: chronological split।
- Test set একবার final evaluation-এর জন্য রাখবে; tuning-এর জন্য নয়।

## ৪. Baseline Model কেন তৈরি করব?

Complex model-এর আগে simple baseline তৈরি করো। Classification-এ DummyClassifier এবং Logistic Regression ভালো starting point হতে পারে।

~~~python
from sklearn.dummy import DummyClassifier

baseline = DummyClassifier(strategy="prior")
baseline.fit(X_train, y_train)
print(baseline.score(X_test, y_test))
~~~

Baseline score দেখে metric meaningful কি না, data-তে signal আছে কি না এবং complex model আসলেই value যোগ করছে কি না বুঝতে পারবে। Final evaluation-এ test set বারবার ব্যবহার না করে validation/CV ব্যবহার করো।

## ৫. Preprocessing Pipeline কীভাবে তৈরি করব?

Numeric ও categorical feature-এর preprocessing আলাদা হতে পারে। `ColumnTransformer` column-specific transformation করে; `Pipeline` preprocessing ও estimator একসঙ্গে রাখে।

~~~python
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
~~~

~~~mermaid
flowchart TD
    A["Raw feature DataFrame"] --> B["ColumnTransformer"]
    B --> C["Numeric pipeline"]
    B --> D["Categorical pipeline"]
    C --> E["Combined transformed features"]
    D --> E
    E --> F["Classifier"]
    F --> G["Prediction"]
~~~

Pipeline leakage risk কমায় কারণ প্রতিটি fit-এর সময় imputer/scaler/encoder শুধু training fold-এ fit হয়। Test data-তে pipeline শুধু learned transformation apply করে।

## ৬. Prediction এবং Evaluation কীভাবে করব?

~~~python
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
~~~

Confusion matrix-এর চারটি অংশ:
- True Positive: positive correctly predict।
- True Negative: negative correctly predict।
- False Positive: negative case-কে positive বলা।
- False Negative: positive case miss করা।

শুধু accuracy দেখে সিদ্ধান্ত নেবে না, বিশেষ করে class imbalance থাকলে। Precision, recall, F1, PR-AUC বা business cost বিবেচনা করো।

## ৭. Threshold tuning কীভাবে করব?

Classifier-এর predict() default threshold ব্যবহার করে class label দেয়। Business objective অনুযায়ী threshold বদলানো যেতে পারে। Threshold validation data-তে select করো, test data-তে নয়।

~~~python
import numpy as np
from sklearn.metrics import precision_recall_curve

validation_probabilities = model.predict_proba(X_validation)[:, 1]
precision, recall, thresholds = precision_recall_curve(
    y_validation,
    validation_probabilities,
)
~~~

Threshold নির্বাচন করার সময় precision/recall trade-off, operational capacity এবং false positive/negative-এর cost বিবেচনা করো। Test set দিয়ে threshold tune কোরো না।

## ৮. Cross Validation কেন ব্যবহার করব?

একটি train/validation split-এর উপর নির্ভর না করে training data-কে একাধিক fold-এ ভাগ করে model evaluate করাকে cross-validation বলে।

~~~python
from sklearn.model_selection import cross_val_score, StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(
    model,
    X_train,
    y_train,
    cv=cv,
    scoring="f1",
)

print(scores.mean(), scores.std())
~~~

StratifiedKFold class proportion ধরে রাখতে পারে। Grouped বা time-series data-তে উপযুক্ত GroupKFold/TimeSeriesSplit ব্যবহার করো। Pipeline থাকলে প্রতিটি fold-এ preprocessing fit হয়।

## ৯. Hyperparameter Tuning কীভাবে করব?

~~~python
from sklearn.model_selection import GridSearchCV

search = GridSearchCV(
    model,
    param_grid={
        "classifier__C": [0.01, 0.1, 1, 10],
    },
    scoring="f1",
    cv=cv,
    n_jobs=-1,
)
search.fit(X_train, y_train)

best_model = search.best_estimator_
print(search.best_params_)
~~~

- GridSearchCV নির্দিষ্ট parameter combination সব পরীক্ষা করে।
- RandomizedSearchCV নির্দিষ্ট distribution থেকে sample নিয়ে পরীক্ষা করে।
- Search-এর scoring metric business goal-এর সঙ্গে মিলতে হবে।
- বড় search space compute cost বাড়ায় এবং overfit to validation selection হতে পারে।

## ১০. Model Calibration কখন দরকার?

Probability output business decision বা risk estimate-এ ব্যবহার করলে calibration গুরুত্বপূর্ণ। Model score ranking ভালো হলেও probability বাস্তব frequency-এর সঙ্গে aligned নাও হতে পারে। Calibration train/CV strategy মেনে fit করো; final test set দিয়ে calibrator fit কোরো না।

Scikit-learn-এ CalibratedClassifierCV ব্যবহার করা যায়। Calibration curve ও Brier score দিয়ে probability reliability inspect করা যায়।

## ১১. Model Persistence কীভাবে করব?

~~~python
import joblib

joblib.dump(best_model, "fraud_model.joblib")
loaded_model = joblib.load("fraud_model.joblib")

predictions_after_reload = loaded_model.predict(X_test)
~~~

শুধু trusted artifact load করবে। Pickle/joblib untrusted source থেকে load করা নিরাপদ নয়, কারণ deserialization arbitrary code execute করতে পারে।

Production-এ শুধু model file নয়, এগুলোও track করো:
- Model version ও training timestamp।
- Dataset/version ও feature schema।
- Python এবং library versions।
- Preprocessing ও model configuration।
- Evaluation metrics ও threshold।
- Artifact checksum/registry metadata।

## ১২. Reproducibility কীভাবে নিশ্চিত করব?

- Random seed নির্দিষ্ট করা।
- Dataset version track করা।
- Feature definition document করা।
- Library version pin করা।
- Training configuration সংরক্ষণ করা।
- Evaluation metric ও split strategy লিখে রাখা।
- Model artifact এবং preprocessing একসঙ্গে version করা।

একই seed সব environment-এ bit-for-bit identical result guarantee করে না; library, hardware এবং parallel computation-ও প্রভাব ফেলতে পারে।

## ১৩. Common Mistakes কী?

1. Split করার আগে পুরো dataset preprocess করা।
2. Test set দিয়ে hyperparameter tune করা।
3. Imbalanced data-তে শুধু accuracy ব্যবহার।
4. Prediction-এর সময় unavailable feature ব্যবহার।
5. Training-serving feature mismatch।
6. Model probability-কে calibrated probability ধরে নেওয়া।
7. Model deploy করে monitoring না করা।
8. Untrusted pickle/joblib artifact load করা।
9. Training এবং inference-এর dependency/schema version mismatch।

## ১৪. Production Monitoring-এ কী দেখব?

- Input schema failure ও missing feature rate।
- Prediction distribution ও class/score drift।
- Latency, throughput এবং error rate।
- Delayed ground truth এলে precision/recall বা business outcome।
- Feature distribution drift এবং concept drift।
- Model version ও rollback readiness।

~~~mermaid
flowchart TD
    A["Deployed model"] --> B["Monitor input and system metrics"]
    B --> C["Monitor prediction distribution"]
    C --> D["Collect delayed ground truth"]
    D --> E["Evaluate real-world performance"]
    E --> F{"Degradation / drift?"}
    F -->|না| B
    F -->|হ্যাঁ| G["Investigate data / model"]
    G --> H["Retrain, validate, or rollback"]
    H --> A
~~~

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Split কখন? | Preprocessing fit করার আগে |
| Stratify কেন? | Class proportion ধরে রাখতে |
| Group/time split কেন? | Dependency ও temporal leakage ঠেকাতে |
| Pipeline কেন? | Preprocessing ও estimator একত্রে; CV leakage কমায় |
| ColumnTransformer? | Column-specific preprocessing |
| Test set কখন? | Final evaluation-এ |
| Cross-validation? | Multiple fold-এ performance estimate |
| GridSearchCV? | Parameter grid search |
| Threshold কোথায় tune? | Validation/CV prediction-এ |
| Calibration কেন? | Probability reliability |
| joblib নিরাপত্তা? | Trusted artifact ছাড়া load নয় |
| Production monitoring? | Data, prediction, latency, error ও outcome |

## Practice

নিজের dataset দিয়ে:

1. Problem, prediction time ও target define করো।
2. Data inspect, leakage check এবং split strategy choose করো।
3. Baseline তৈরি করো।
4. Numeric/categorical preprocessing pipeline তৈরি করো।
5. Logistic Regression train করো।
6. Confusion matrix, precision, recall, F1 এবং ROC-AUC/PR-AUC বের করো।
7. Cross-validation চালাও।
8. Hyperparameter tuning করো।
9. Validation data-তে threshold select করো।
10. Final test set-এ একবার evaluate করো।
11. Model save ও reload করে prediction consistency check করো।
12. Feature schema, library version এবং evaluation result record করো।

## Practice করার পদ্ধতি

1. Split strategy কেন বেছে নিয়েছ explain করো।
2. Preprocessing fit কোন data-তে হচ্ছে verify করো।
3. Baseline-এর সঙ্গে candidate model compare করো।
4. Metric ও threshold business objective-এর সঙ্গে align করো।
5. Final test set tuning-এর বাইরে রাখো।
6. Deployment-এর পরে কী monitor করবে তা লিখে রাখো।
