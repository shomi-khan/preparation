# Day 3 Practical Exercises

## Exercise 1: Confusion Matrix
একটি binary classifier-এর confusion matrix তৈরি করো। TP=80, FP=20, FN=10, TN=890 ধরে accuracy, precision, recall, specificity ও F1 হিসাব করো।

## Exercise 2: Threshold Selection
Validation probability দিয়ে 0.2, 0.4, 0.5, 0.7 threshold evaluate করো। Precision/recall table তৈরি করো। কোন threshold business use case-এ ভালো এবং কেন?

## Exercise 3: Imbalanced Classification
Fraud-like imbalanced dataset-এ DummyClassifier ও LogisticRegression compare করো। PR-AUC report করো।

## Exercise 4: Leakage Audit
একটি preprocessing workflow audit করো। Imputation, scaling, feature selection ও oversampling কোথায় fit হচ্ছে তা লিখে leakage আছে কি না দেখো।

## Exercise 5: SQL Analytics
উল্লিখিত customers ও transactions schema-তে:
- Customer lifetime revenue।
- Monthly successful revenue।
- Customer-এর latest transaction।
- Month-wise top 3 customer।
- Zero-transaction customer।
প্রতিটির query লেখো।

## Exercise 6: Window Functions
ROW_NUMBER, RANK, DENSE_RANK, LAG ব্যবহার করে example query লেখো। তাদের পার্থক্য ব্যাখ্যা করো।

## Exercise 7: Query Plan
একটি aggregate query-তে EXPLAIN ANALYZE চালাও। Index-এর আগে/পরে plan ও execution time তুলনা করো (নিজের local test database-এ)।

## Deliverables
- Metric calculation notebook।
- SQL query file।
- Leakage audit note।
- ১ পৃষ্ঠার evaluation summary।

## Self-review
- [ ] Imbalanced metric সঠিকভাবে বেছে নিয়েছি।
- [ ] Threshold validation data-তে tune করেছি।
- [ ] Test set untouched রেখেছি।
- [ ] Temporal split দরকার কি না বিবেচনা করেছি।
- [ ] SQL NULL semantics বুঝেছি।
- [ ] JOIN ও window query নিজে লিখেছি।
- [ ] Query plan পড়েছি।
