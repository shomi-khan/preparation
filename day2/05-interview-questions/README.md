# Day 2 Interview Questions and Answers

## Core Concepts

### ১. Machine Learning কী?

Data থেকে pattern শেখা model ব্যবহার করে prediction বা decision নেওয়ার পদ্ধতি। Training-এ model parameters শেখে এবং inference-এ নতুন input থেকে output তৈরি করে।

### ২. Supervised ও Unsupervised Learning-এর পার্থক্য কী?

Supervised learning-এ labeled data থাকে; unsupervised learning-এ explicit target label থাকে না। Classification/regression supervised task, clustering unsupervised task।

### ৩. Classification এবং Regression-এর পার্থক্য কী?

Classification category predict করে; regression continuous numeric value predict করে। Fraud/legitimate classification, house price regression।

### ৪. Feature ও Label কী?

Feature হলো model input; label হলো target output যা model predict করতে শেখে।

### ৫. Training, Validation ও Test set কেন আলাদা?

Training model fit করে, validation model/hyperparameter selection-এ ব্যবহৃত হয়, test final generalization estimate দেয়। Test বারবার ব্যবহার করলে evaluation optimistic হতে পারে।

### ৬. Overfitting কীভাবে বুঝবে?

Training score অনেক ভালো কিন্তু validation/test score তুলনামূলক খারাপ হলে overfitting-এর ইঙ্গিত। Cross-validation ও learning curve সহায়ক।

### ৭. Underfitting কী?

Model training data-তেও যথেষ্ট pattern ধরতে পারে না। Feature, model capacity বা training strategy উন্নত করতে হতে পারে।

### ৮. Bias-Variance trade-off কী?

High bias অতিরিক্ত simple model-এর systematic error; high variance data change-এ model-এর prediction অস্থিতিশীল হওয়া। লক্ষ্য হলো unseen data-তে ভালো generalization।

### ৯. Data Leakage কী?

Training/evaluation-এ এমন তথ্য ঢুকে যাওয়া যা prediction time-এ পাওয়া সম্ভব নয়, অথবা test data-র তথ্য training process-এ প্রভাব ফেলা। এটি evaluation score কৃত্রিমভাবে বাড়াতে পারে।

### ১০. Feature Engineering কী?

Raw data থেকে useful feature তৈরি বা transform করা, যেমন timestamp থেকে hour, transaction history থেকে rolling average।

## Algorithms

### ১১. Linear Regression কী করে?

Features-এর weighted linear combination দিয়ে continuous target estimate করে। এটি simple baseline ও interpretable relationship-এর জন্য উপকারী।

### ১২. Logistic Regression কি regression?

নাম regression হলেও এটি classification-এর জন্য ব্যবহৃত হয়। Sigmoid/softmax-এর মাধ্যমে class probability estimate করতে পারে।

### ১৩. Decision Tree কীভাবে কাজ করে?

Feature ও threshold দিয়ে data-কে recursiveভাবে split করে, যাতে node-এ target আরও homogeneous হয়। গভীর tree overfit করতে পারে।

### ১৪. Random Forest কী?

Randomized training data/features ব্যবহার করে অনেক decision tree train করে এবং prediction aggregate করে। এটি variance কমাতে পারে।

### ১৫. Bagging ও Boosting-এর পার্থক্য কী?

Bagging-এ model সাধারণত parallel/independently train হয় এবং output aggregate করা হয়। Boosting sequentialভাবে আগের model-এর error address করে নতুন learner যোগ করে।

### ১৬. KNN-এর limitation কী?

Inference ব্যয়বহুল হতে পারে, scaling ও distance metric গুরুত্বপূর্ণ, high-dimensional data-তে distance less informative হতে পারে।

### ১৭. K-Means-এর limitation কী?

K আগে নির্ধারণ করতে হয়, initialization-sensitive হতে পারে, outlier-এ প্রভাবিত হয় এবং non-spherical cluster-এ খারাপ কাজ করতে পারে।

### ১৮. Regularization কেন দরকার?

Model complexity নিয়ন্ত্রণ করে overfitting কমাতে। L1 sparse coefficient তৈরি করতে পারে; L2 coefficient shrink করে।

### ১৯. Feature scaling কখন জরুরি?

Distance-based ও gradient-based model যেমন KNN, SVM, Logistic Regression-এর ক্ষেত্রে সাধারণত গুরুত্বপূর্ণ। Tree-based model সাধারণত scaling-এর উপর কম নির্ভরশীল।

## Workflow

### ২০. Baseline model কেন?

পরবর্তী model সত্যিকারের উন্নতি করেছে কি না তুলনা করার reference দেয়। DummyClassifier বা simple Logistic Regression দিয়ে শুরু করা যায়।

### ২১. Scikit-learn Pipeline কেন ব্যবহার করবে?

Preprocessing ও estimator-কে এক workflow-তে রাখে, cross-validation-এ প্রতিটি fold-এ preprocessing সঠিকভাবে fit করতে সাহায্য করে এবং leakage risk কমায়।

### ২২. `fit_transform` ও `transform`-এর পার্থক্য কী?

`fit` data থেকে parameter শেখে; `transform` সেই parameter ব্যবহার করে data transform করে। Training data-তে fit/transform, test data-তে শুধু transform করতে হবে।

### ২৩. Cross-validation কী?

Training data-কে একাধিক fold-এ ভাগ করে বিভিন্ন অংশে train/validate করে performance estimate করা। Data size ও split strategy অনুযায়ী উপযুক্ত CV বেছে নিতে হয়।

### ২৪. Hyperparameter কী?

Training-এর আগে নির্ধারিত configuration, যেমন tree depth বা Logistic Regression-এর C। Model parameter training data থেকে শেখে।

### ২৫. Model persistence-এ কী সতর্কতা?

Trusted artifact ছাড়া pickle/joblib load নয়। Model, preprocessing, feature schema, library version ও training metadata একসঙ্গে version করতে হবে।

## Scenario Questions

### ২৬. Fraud dataset-এ ৯৯% transaction legitimate। Accuracy ৯৯% হলে কি model ভালো?

অবশ্যই নয়। সব transaction legitimate বললেও ৯৯% accuracy হতে পারে। Confusion matrix, fraud-class precision/recall, F1 ও business cost দেখতে হবে।

### ২৭. Training score ৯৮%, test score ৭০%। কী করবে?

Overfitting বা train/test distribution mismatch investigate করব। Cross-validation, simpler model, regularization, leakage check, feature quality ও split strategy পর্যালোচনা করব।

### ২৮. Test score খুব ভালো কিন্তু production performance খারাপ। সম্ভাব্য কারণ?

Data leakage, distribution shift, training-serving skew, incorrect labels, production preprocessing mismatch, delayed feedback বা metric mismatch।

### ২৯. Model probability ০.৮ দিলে কি ৮০% নিশ্চিত?

না। এটি model score/probability estimate। Calibration না থাকলে predicted probability বাস্তব frequency-এর সঙ্গে নাও মিলতে পারে।

### ৩০. কোন model বেছে নেবে?

একটি metric দেখে নয়। Baseline, cross-validation, business objective, latency, interpretability, training/inference cost, data size ও maintenance বিবেচনা করব।

## Coding Questions

### ৩১. Train/test split কীভাবে করবে?

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
```

### ৩২. Logistic Regression fit করে prediction কীভাবে করবে?

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

### ৩৩. Confusion matrix কীভাবে বের করবে?

```python
from sklearn.metrics import confusion_matrix
print(confusion_matrix(y_test, predictions))
```

### ৩৪. Precision ও Recall কী?

Precision = TP / (TP + FP)। Predicted positive-এর মধ্যে কতগুলো সত্য positive।

Recall = TP / (TP + FN)। Actual positive-এর মধ্যে কতগুলো শনাক্ত হয়েছে।

### ৩৫. Pipeline কেন ব্যবহার করবে?

যাতে preprocessing training fold-এর data থেকে fit হয় এবং একই transformation inference-এও প্রয়োগ হয়। এটি reproducibility ও leakage prevention-এ সাহায্য করে।

## Oral Practice

প্রতিটি প্রশ্নের উত্তর ৬০-৯০ সেকেন্ডে বলার অনুশীলন করো। Definition-এর পর একটি বাস্তব উদাহরণ, limitation এবং trade-off যোগ করো।


---

## Additional Interview Questions: Model Selection, Evaluation and Production

### ৩৬. Semi-supervised এবং self-supervised learning-এর পার্থক্য কী?

Semi-supervised learning অল্প labeled data ও বেশি unlabeled data ব্যবহার করে। Self-supervised learning data থেকেই training target তৈরি করে, যেমন text-এর পরের token predict করা। দুটোর supervision source আলাদা।

### ৩৭. Reinforcement Learning কীভাবে supervised learning থেকে আলাদা?

Supervised learning-এ training example-এর expected label থাকে। Reinforcement learning-এ agent environment-এ action নেয় এবং reward/feedback পায়; reward maximize করার policy শেখে।

### ৩৮. কখন random split ব্যবহার করা উচিত নয়?

যখন data-তে temporal dependency, repeated entity/group বা sequence dependency আছে। Time-series-এ chronological split; repeated patient/customer-এর ক্ষেত্রে group-aware split দরকার হতে পারে। Split strategy real prediction scenario reflect করবে।

### ৩৯. Cross-validation-এর score-এর mean-এর পাশাপাশি standard deviation কেন দেখা দরকার?

Mean overall expected performance-এর ধারণা দেয়; standard deviation fold-to-fold variability বোঝায়। Variability বেশি হলে model বা dataset split-sensitive হতে পারে। তবে fold score independent sample নয়, তাই interpretation সতর্কভাবে করতে হবে।

### ৪০. Loss function এবং evaluation metric-এর পার্থক্য কী?

Loss function training-এ optimize করা objective। Evaluation metric model performance communicate/compare করতে ব্যবহৃত হয়। Training log loss optimize করলেও business objective precision, recall বা expected cost হতে পারে।

### ৪১. Class imbalance-এ accuracy misleading কেন?

যদি 99% sample negative হয়, সব sample negative predict করলেও accuracy 99% হতে পারে। Confusion matrix, minority-class precision/recall, F1, PR-AUC এবং error cost দেখতে হবে।

### ৪২. Precision এবং Recall-এর trade-off কী?

Precision = TP/(TP+FP): predicted positive-এর মধ্যে কতটি সত্য positive। Recall = TP/(TP+FN): actual positive-এর মধ্যে কতটি detect হয়েছে। Threshold বদলালে সাধারণত একটির উন্নতিতে অন্যটি কমতে পারে।

### ৪৩. ROC-AUC এবং PR-AUC কখন দেখবে?

ROC-AUC positive ও negative pair ranking quality-এর ধারণা দেয়। Highly imbalanced positive class-এ PR curve precision/recall trade-off স্পষ্ট করতে পারে। কোন metric উপযুক্ত তা class prevalence ও business cost-এর উপর নির্ভর করে।

### ৪৪. Model threshold কীভাবে select করবে?

Validation data বা cross-validation-এর out-of-fold probability দিয়ে threshold নির্বাচন করবে। Precision/recall target, expected cost, review capacity বা business constraint বিবেচনা করবে। Test set দিয়ে threshold tune করবে না।

### ৪৫. Probability calibration কী?

Model-এর predicted probability observed frequency-এর সঙ্গে কতটা মেলে সেটি calibration। Model ranking ভালো হলেও probability overconfident/underconfident হতে পারে। Calibration curve ও Brier score ব্যবহার করা যায়।

### ৪৬. SVM কখন ব্যবহার করবে?

SVM margin-based decision boundary শেখে; kernel দিয়ে nonlinear boundary model করা যায়। Feature scaling গুরুত্বপূর্ণ। Medium-sized/high-dimensional dataset-এ উপযোগী হতে পারে, কিন্তু বড় dataset-এ training cost বেশি হতে পারে।

### ৪৭. Naive Bayes-এর naive assumption কী?

প্রতিটি class condition-এ features conditionally independent ধরে। বাস্তবে assumption পুরোপুরি সত্য না হলেও text classification-এর মতো problem-এ দ্রুত baseline হিসেবে ভালো কাজ করতে পারে।

### ৪৮. Gradient Boosting কীভাবে কাজ করে?

Sequentially weak learner যোগ করে আগের ensemble-এর loss/residual কমানোর চেষ্টা করে। Random Forest-এর মতো independent tree aggregation নয়; boosting stage-গুলো sequential dependency রাখে।

### ৪৯. Hyperparameter এবং model parameter-এর পার্থক্য কী?

Model parameter training data থেকে শেখে, যেমন linear model-এর coefficient। Hyperparameter training-এর আগে বা tuning-এ নির্ধারিত হয়, যেমন tree depth, learning rate বা regularization strength।

### ৫০. Feature scaling কেন Pipeline-এর ভিতরে রাখতে হবে?

Cross-validation-এর প্রতিটি fold-এ scaler শুধু training fold থেকে mean/standard deviation শেখে। Pipeline না থাকলে full dataset দিয়ে scaler fit করে validation fold-এর information training preprocessing-এ leak হতে পারে।

### ৫১. Production-এ model performance কীভাবে monitor করবে?

Input schema failure, missing rate, feature/prediction distribution, latency, error rate এবং delayed ground-truth metric monitor করবে। Data drift ও concept drift আলাদা করে investigate করবে। Model version ও rollback path রাখতে হবে।

### ৫২. Training-serving skew কী?

Training ও production inference-এ feature calculation, missing-value handling, encoding বা schema আলাদা হলে model ভিন্ন input distribution পায়। Shared preprocessing artifact/pipeline ও feature contract দিয়ে mismatch কমানো যায়।

### ৫৩. Batch learning এবং online learning-এর পার্থক্য কী?

Batch learning নির্দিষ্ট dataset দিয়ে model train করে এবং periodic retraining করে। Online learning নতুন data আসার সঙ্গে incremental update করতে পারে। Online setup-এ drift, noisy feedback, rollback ও stability risk বেশি সতর্কভাবে manage করতে হয়।

## Model Evaluation Flow

~~~mermaid
flowchart TD
    A["Define business objective"] --> B["Choose evaluation metric"]
    B --> C["Create leakage-aware split"]
    C --> D["Train baseline"]
    D --> E["Cross-validation and model selection"]
    E --> F["Tune threshold / calibrate on validation"]
    F --> G["Freeze model"]
    G --> H["Final test evaluation"]
    H --> I["Production monitoring"]
~~~

## Oral Practice

প্রতিটি প্রশ্নের উত্তর ৬০-৯০ সেকেন্ডে বলার অনুশীলন করো। Definition-এর পর একটি বাস্তব উদাহরণ, limitation এবং trade-off যোগ করো। Model choice বা metric choice-এর পেছনে business reasoning explain করো।
