# Day 2 Practical Exercises

## Exercise 1: Problem Classification

নিচের প্রতিটি problem classification, regression নাকি clustering, তা চিহ্নিত করো এবং কারণ লেখো:

1. Email spam detection।
2. আগামী মাসের electricity demand predict।
3. Customer segmentation।
4. Loan default risk prediction।
5. Used car price estimation।

## Exercise 2: Data Exploration

একটি tabular dataset বেছে নাও। বের করো:
- Row/column count।
- Data types।
- Missing value count।
- Duplicate row count।
- Target class distribution।
- Numeric feature summary।

## Exercise 3: Train/Test Split

একটি classification dataset-এ stratified split করো। Split-এর আগে ও পরে class distribution compare করো। কেন stratification দরকার তা লিখো।

## Exercise 4: Baseline Model

DummyClassifier এবং LogisticRegression train করো। Accuracy নয়, F1 ও confusion matrix তুলনা করো। Baseline-এর চেয়ে উন্নতি হয়েছে কি?

## Exercise 5: Model Comparison

একই preprocessing ও split strategy ব্যবহার করে:
- Logistic Regression
- Decision Tree
- Random Forest

তুলনা করো। Model complexity, training time এবং evaluation score নোট করো।

## Exercise 6: Leakage Demonstration

একটি dataset-এর পুরো data-তে scaler fit করে split করার পদ্ধতি এবং Pipeline-এর মাধ্যমে split-এর পরে scaler fit করার পদ্ধতি তুলনা করো। Leakage কেন হতে পারে ব্যাখ্যা করো।

## Exercise 7: Imbalanced Dataset

একটি imbalanced classification dataset নাও। Accuracy, precision, recall ও F1 তুলনা করো। Fraud detection-এর মতো use case-এ false negative-এর cost ব্যাখ্যা করো।

## Exercise 8: Model Artifact

Best model joblib দিয়ে save করো, reload করো এবং held-out sample-এ prediction compare করো। Model version ও dependency version লিখে রাখো।

## Deliverables

- একটি notebook বা Python project।
- Dataset সম্পর্কে ছোট description।
- Model comparison table।
- Confusion matrix ও metric report।
- Leakage prevention-এর ব্যাখ্যা।
- ৩-৫ মিনিটের project walkthrough।

## Self-review

- [ ] Target ও features সঠিকভাবে চিহ্নিত করেছি।
- [ ] Train/test split করেছি।
- [ ] Preprocessing Pipeline-এ রেখেছি।
- [ ] Baseline-এর সঙ্গে compare করেছি।
- [ ] উপযুক্ত metric বেছে নিয়েছি।
- [ ] Test set tuning-এ ব্যবহার করিনি।
- [ ] Limitation ও next step লিখেছি।
