# Day 3 Practical Exercises

Day 3-এর লক্ষ্য model-এর score report করা নয়; evaluation কেন বিশ্বাসযোগ্য, কোন error গুরুত্বপূর্ণ এবং data analysis/SQL দিয়ে কীভাবে সেই ফলাফল validate করবে তা শেখা।

## Exercise 1: Confusion Matrix Calculation

একটি binary classifier-এর confusion matrix তৈরি করো। TP=80, FP=20, FN=10, TN=890 ধরে হিসাব করো:
- Accuracy
- Precision
- Recall
- Specificity
- F1

তারপর প্রতিটি metric-এর business interpretation লেখো। কোন metric একা যথেষ্ট নয় তা explain করো।

## Exercise 2: Threshold Selection

Validation probability দিয়ে 0.2, 0.4, 0.5, 0.7 threshold evaluate করো। Precision/recall/F1 table তৈরি করো।

**Tasks**
1. Threshold অনুযায়ী confusion matrix তৈরি করো।
2. কোন threshold business use case-এ ভালো এবং কেন?
3. Review team দিনে সর্বোচ্চ ৫০টি case investigate করতে পারলে threshold selection কীভাবে বদলাবে?
4. Test set ব্যবহার না করে decision justify করো।

## Exercise 3: Imbalanced Classification

Fraud-like imbalanced dataset-এ DummyClassifier ও LogisticRegression compare করো।

- Class distribution report করো।
- Accuracy, precision, recall, F1 ও average precision/PR-AUC report করো।
- Class weight experiment করো।
- Validation data-তে threshold tune করো।
- False negative ও false positive cost explain করো।

## Exercise 4: Regression Evaluation

একটি regression dataset-এ MAE, MSE, RMSE এবং R² বের করো।

**Tasks**
- Outlier যোগ করে metric কীভাবে বদলায় observe করো।
- MAE ও RMSE-এর unit explain করো।
- Negative R² কী বোঝায় explain করো।
- Residual plot inspect করো।

## Exercise 5: Calibration Check

একটি probabilistic classifier-এর reliability diagram ও Brier score বের করো। Calibration দরকার কি না business use case দিয়ে justify করো। Calibration শুধু training/CV data থেকে fit করো।

## Exercise 6: Leakage Audit

একটি preprocessing workflow audit করো। Imputation, scaling, feature selection ও oversampling কোথায় fit হচ্ছে তা লিখে leakage আছে কি না দেখো।

~~~mermaid
flowchart TD
    A["Raw data"] --> B["Split train / validation / test"]
    B --> C["Fit preprocessing on train only"]
    C --> D["Transform validation / test"]
    D --> E["Train and evaluate"]
    E --> F["Keep final test untouched"]
~~~

## Exercise 7: SQL Analytics

Customers ও transactions schema-তে:
- Customer lifetime revenue।
- Monthly successful revenue।
- Customer-এর latest transaction।
- Month-wise top 3 customer।
- Zero-transaction customer।
- Duplicate transaction ID।
প্রতিটির query লেখো।

## Exercise 8: Window Functions

ROW_NUMBER, RANK, DENSE_RANK, LAG ব্যবহার করে example query লেখো। Tie থাকলে ranking কীভাবে বদলায় তা sample data দিয়ে explain করো।

## Exercise 9: SQL NULL and JOIN Semantics

Sample table-এ NULL, duplicate join key এবং unmatched record যোগ করো।

- COUNT(*) বনাম COUNT(column)।
- INNER বনাম LEFT JOIN।
- LEFT JOIN-এর পরে WHERE filter result বদলায় কি না।
- EXISTS বনাম IN।
- Join cardinality validation।

## Exercise 10: Query Plan and Index

একটি aggregate query-তে EXPLAIN ANALYZE চালাও (নিজের local test database-এ)।

- Index-এর আগে/পরে plan ও execution time তুলনা করো।
- Composite index-এর column order পরীক্ষা করো।
- Write overhead এবং storage cost note করো।
- Small table-এ sequential scan কেন যুক্তিযুক্ত হতে পারে explain করো।

## Exercise 11: SQL Safety and Transactions

1. Parameterized query দিয়ে user input filter করো।
2. Dynamic sort column allowlist দিয়ে select করো।
3. Transaction দিয়ে debit/credit pair atomicভাবে update করো।
4. Failure ঘটিয়ে rollback behavior পরীক্ষা করো।
5. ACID-এর প্রতিটি property example দিয়ে explain করো।

## Exercise 12: Data Quality Audit

একটি transaction dataset-এ:
- Missing value।
- Duplicate ID।
- Invalid amount।
- Malformed timestamp।
- Unmatched customer key।
- Temporal inconsistency।

প্রতিটি issue count করো, handling policy লিখো এবং accepted + rejected row count reconcile করো।

## Deliverables

- Metric calculation notebook।
- SQL query file।
- Leakage audit note।
- Data quality report।
- ১ পৃষ্ঠার evaluation summary।
- Query plan screenshot বা text output।
- ৩-৫ মিনিটের walkthrough।

## Self-review

- [ ] Imbalanced metric সঠিকভাবে বেছে নিয়েছি।
- [ ] Threshold validation data-তে tune করেছি।
- [ ] Test set untouched রেখেছি।
- [ ] Temporal/group split দরকার কি না বিবেচনা করেছি।
- [ ] SQL NULL semantics বুঝেছি।
- [ ] JOIN cardinality verify করেছি।
- [ ] Window query নিজে লিখেছি।
- [ ] Query plan পড়েছি।
- [ ] Parameterized query ব্যবহার করেছি।
- [ ] Row-count reconciliation করেছি।

## Completion Criteria

1. Metric formula ও business meaning explain করতে পারি।
2. Threshold ও calibration test set ছাড়া নির্বাচন করেছি।
3. Leakage risk শনাক্ত ও prevent করেছি।
4. SQL query NULL, duplicate ও tie case-এ সঠিক।
5. Query performance actual execution plan দিয়ে analyze করেছি।
6. Data quality decision এবং rejected row reason document করেছি।
