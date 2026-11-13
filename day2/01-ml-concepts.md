# Machine Learning Concepts

## ১. Machine Learning কী?

Machine Learning (ML) হলো এমন পদ্ধতি যেখানে computer explicit rule প্রতিটি ক্ষেত্রে হাতে লিখে না দিয়ে data থেকে pattern শেখে এবং সেই pattern ব্যবহার করে prediction বা decision নেয়।

উদাহরণ: আগের transaction-এর features দেখে নতুন transaction fraud হওয়ার সম্ভাবনা অনুমান করা।

## ২. Traditional Programming বনাম Machine Learning

Traditional programming-এ input ও hand-written rules দিয়ে output তৈরি হয়। ML-এ training data থেকে model parameters শেখা হয়; inference-এর সময় নতুন input model-এ দিলে prediction পাওয়া যায়।

```text
Traditional: Input + Rules -> Output
ML Training: Input + Expected Output -> Learned Model
ML Inference: New Input + Learned Model -> Prediction
```

## ৩. Supervised Learning কী?

যে training dataset-এ input-এর পাশাপাশি expected answer বা label থাকে, সেটি supervised learning।

দুটি প্রধান task:
- **Classification:** category predict করা, যেমন fraud / legitimate।
- **Regression:** continuous numeric value predict করা, যেমন house price।

## ৪. Unsupervised Learning কী?

যে data-তে explicit target label নেই, সেখানে structure বা pattern খোঁজার পদ্ধতি। যেমন clustering, dimensionality reduction ও anomaly exploration।

## ৫. Classification বনাম Regression

| বিষয় | Classification | Regression |
|---|---|---|
| Output | Class/category | Continuous number |
| উদাহরণ | Spam / Not spam | আগামী মাসের sales |
| Common metric | Precision, Recall, F1 | MAE, MSE, RMSE, R² |

## ৬. Feature এবং Label কী?

- **Feature (X):** model-এর input variable।
- **Label/Target (y):** model যে outcome predict করতে শেখে।

Fraud detection-এ amount, merchant category, time, device age feature হতে পারে; fraud status label হতে পারে।

Feature যেন prediction করার সময় বাস্তবে available থাকে। ভবিষ্যতের তথ্য feature-এ ঢুকে গেলে leakage হতে পারে।

## ৭. Training, Validation এবং Test Set

- **Training set:** model parameters শেখে।
- **Validation set:** model/ hyperparameter নির্বাচন ও tuning-এ ব্যবহৃত হয়।
- **Test set:** final unbiased performance estimate-এর জন্য একেবারে শেষে ব্যবহৃত হয়।

Test set বারবার দেখে model tune করলে সেটিও effectively training process-এর অংশ হয়ে যায়।

## ৮. Train/Test Split কেন?

Training data-তে ভালো performance মানেই unseen data-তে ভালো performance নয়। Data ভাগ করে generalization যাচাই করা হয়।

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)
```

Classification-এ `stratify=y` class proportion কাছাকাছি রাখতে সাহায্য করে।

## ৯. Overfitting এবং Underfitting

- **Overfitting:** training data-র noise-সহ pattern শিখে ফেলে; training performance ভালো, unseen performance খারাপ।
- **Underfitting:** model যথেষ্ট pattern ধরতে পারে না; training ও test উভয় performance খারাপ।

সমাধান হিসেবে model complexity, regularization, feature quality, more data ও cross-validation বিবেচনা করা যায়।

## ১০. Bias-Variance Trade-off

- High bias: model অতিরিক্ত simple, systematic error বেশি।
- High variance: training data পরিবর্তনে model prediction অনেক বদলে যায়।

উদ্দেশ্য হলো unseen data-তে ভালো generalization, শুধু training score বাড়ানো নয়।

## ১১. Feature Engineering কী?

Raw data থেকে useful input variable তৈরি বা transform করা।

উদাহরণ:
- Timestamp থেকে hour, weekday।
- Date থেকে account age।
- Transaction history থেকে rolling average।
- Text থেকে TF-IDF features।

Feature তৈরি করার সময় target leakage এবং train/test boundary মানতে হবে।

## ১২. Data Leakage কী?

যখন training বা evaluation-এর সময় এমন তথ্য model পায় যা বাস্তব prediction time-এ পাওয়া সম্ভব নয়, অথবা test data-র information training process-এ ঢুকে যায়।

উদাহরণ: পুরো dataset-এর mean দিয়ে missing value fill করে তারপর split করা। সঠিকভাবে preprocessing শেখা উচিত training data-তে, তাই Scikit-learn Pipeline ব্যবহার করা ভালো।

## ১৩. Baseline Model কেন?

Baseline হলো simple reference model, যার সঙ্গে পরবর্তী model তুলনা করা যায়। Classification-এ DummyClassifier বা Logistic Regression দিয়ে শুরু করা যায়। Baseline ছাড়া জটিল model আসলেই উন্নতি করেছে কি না বোঝা কঠিন।

## ১৪. ML Lifecycle

```text
Problem Definition
  -> Data Collection
  -> Data Validation / Cleaning
  -> Feature Engineering
  -> Train / Validation
  -> Model Evaluation
  -> Deployment
  -> Monitoring
  -> Retraining / Improvement
```

## Practice Questions

1. ML model আর rule-based system-এর পার্থক্য কী?
2. Supervised learning-এ label কেন দরকার?
3. Classification ও regression-এর output কীভাবে আলাদা?
4. Test set দিয়ে বারবার tuning করলে কী সমস্যা?
5. Data leakage-এর একটি বাস্তব উদাহরণ দাও।
6. Baseline model কেন তৈরি করবে?
