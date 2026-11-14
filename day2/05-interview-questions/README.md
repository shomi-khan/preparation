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
