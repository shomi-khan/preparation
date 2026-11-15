# Day 3 Interview Questions and Answers

## Evaluation

### ১. Confusion Matrix কী?
Classification prediction-কে TP, TN, FP ও FN-এ ভাগ করে। এটি ভুলের ধরন বোঝায়।

### ২. Accuracy কখন misleading?
Class imbalance থাকলে majority class predict করেই high accuracy পাওয়া যায়। Minority class performance বোঝা যায় না।

### ৩. Precision কী?
Predicted positive-এর মধ্যে সত্য positive-এর অনুপাত: TP/(TP+FP)। False positive cost বেশি হলে গুরুত্বপূর্ণ।

### ৪. Recall কী?
Actual positive-এর মধ্যে সঠিক positive: TP/(TP+FN)। False negative cost বেশি হলে গুরুত্বপূর্ণ।

### ৫. Precision ও Recall-এর trade-off কী?
Threshold কমালে সাধারণত বেশি positive predict হয়, recall বাড়তে ও precision কমতে পারে। Threshold business cost অনুযায়ী বেছে নিতে হয়।

### ৬. F1 কেন harmonic mean?
Precision বা recall-এর যেকোনো একটি খুব কম হলে F1-ও কমে। তবে F1 true negative ও business cost সরাসরি বিবেচনা করে না।

### ৭. ROC-AUC কী বোঝায়?
Positive sample-কে negative sample-এর চেয়ে বেশি score দেওয়ার ranking ability-এর একটি measure। এটি নির্দিষ্ট threshold-এর performance নয়।

### ৮. PR-AUC কখন বেশি উপকারী?
Positive class rare হলে precision-recall curve minority positive class-এর performance সম্পর্কে বেশি informative হতে পারে।

### ৯. Cross-validation কেন?
একটি নির্দিষ্ট split-এর উপর নির্ভরতা কমিয়ে training data-তে একাধিক train/validation split-এর performance estimate পেতে।

### ১০. Time series-এ random split কেন সমস্যা?
Future observation training-এ চলে আসতে পারে এবং বাস্তব deployment-এর chronology ভেঙে যায়। Time-based split ব্যবহার করতে হয়।

### ১১. Calibration কী?
Predicted probability বাস্তব event frequency-এর সঙ্গে সামঞ্জস্যপূর্ণ কি না। Risk score-ভিত্তিক decision-এ গুরুত্বপূর্ণ।

### ১২. Data drift ও concept drift কী?
Data drift input distribution পরিবর্তন; concept drift input-target relationship পরিবর্তন।

## SQL

### ১৩. WHERE ও HAVING-এর পার্থক্য?
WHERE row-level filter, GROUP BY-এর আগে। HAVING aggregate group filter করে।

### ১৪. INNER JOIN ও LEFT JOIN?
INNER কেবল match রাখা row দেয়। LEFT left table-এর সব row রাখে; unmatched right column NULL।

### ১৫. UNION ও UNION ALL?
UNION duplicate remove করে; UNION ALL duplicate রেখে দেয় এবং সাধারণত কম কাজ করে।

### ১৬. COUNT(*) ও COUNT(column)?
COUNT(*) সব row গোনে; COUNT(column) NULL বাদ দেয়।

### ১৭. NULL = NULL কি true?
না। SQL NULL unknown semantics; NULL check করতে IS NULL ব্যবহার করতে হয়।

### ১৮. CTE কী?
WITH clause দিয়ে named query expression তৈরি করা হয়, যা complex query readable ও modular করে।

### ১৯. Window Function কী?
একটি row-এর সঙ্গে related row group-এর উপর calculation করে কিন্তু output row collapse করে না।

### ২০. ROW_NUMBER, RANK ও DENSE_RANK?
ROW_NUMBER unique sequential number দেয়; RANK tie হলে gap রাখে; DENSE_RANK tie হলে gap রাখে না।

### ২১. Index কী এবং trade-off কী?
Index lookup দ্রুত করতে পারে, কিন্তু storage নেয় এবং insert/update/delete ব্যয় বাড়াতে পারে।

### ২২. Query optimize করবে কীভাবে?
EXPLAIN plan দেখব, selective filter, suitable index, join cardinality, unnecessary scan ও SELECT * পরীক্ষা করব। আগে measure করব।

### ২৩. SQL Injection কীভাবে ঠেকাবে?
Parameterized query ব্যবহার করব; user value string concatenate করব না। Dynamic identifiers allowlist করব।

## Scenario

### ২৪. Fraud recall বেশি, precision কম। কী করবে?
False positive-এর operational cost, review capacity ও fraud loss তুলনা করব। Validation data-তে threshold tune, better features ও model calibration পরীক্ষা করব।

### ২৫. Production metric কমে গেলে?
Data quality, schema change, drift, training-serving skew, label delay, threshold ও infrastructure failure আলাদা করে investigate করব।

### ২৬. Query ধীর হলে?
EXPLAIN ANALYZE, index, filter selectivity, join plan, row count estimate ও data volume inspect করব। Index blindly যোগ করব না।

### ২৭. প্রতি customer-এর latest transaction কীভাবে বের করবে?
ROW_NUMBER window function দিয়ে customer partition করে created_at descending rank 1 filter করতে পারি।

## Calculation Practice
TP=80, FP=20, FN=10, TN=890 হলে:
- Precision = 80/100 = 0.80
- Recall = 80/90 ≈ 0.889
- Accuracy = 970/1000 = 0.97
- F1 ≈ 0.842
নিজে হিসাব করে যাচাই করো।


---

## Additional Interview Questions: Evaluation Depth and SQL Reliability

### ২৮. Specificity কী?

Specificity = TN/(TN+FP)। Actual negative sample-এর মধ্যে কতগুলো সঠিকভাবে negative predict হয়েছে। False positive cost বেশি হলে specificity গুরুত্বপূর্ণ।

### ২৯. ROC-AUC এবং PR-AUC-এর পার্থক্য কী?

ROC-AUC TPR বনাম FPR-এর threshold sweep summarize করে। PR-AUC precision বনাম recall trade-off summarize করে। Positive class খুব rare হলে PR-AUC/average precision minority-class performance বোঝাতে বেশি informative হতে পারে।

### ৩০. Average Precision কি trapezoidal PR-AUC-এর সঙ্গে একই?

সবসময় নয়। Scikit-learn-এর average_precision_score precision-recall points-এর weighted sum ব্যবহার করে; trapezoidal integration-এর result আলাদা হতে পারে। Report-এ কোন metric ব্যবহার করেছ স্পষ্ট করো।

### ৩১. Regression model কীভাবে evaluate করবে?

MAE average absolute error; MSE squared error average; RMSE MSE-এর square root, target-এর unit-এ; R² mean baseline-এর তুলনায় explained variance-এর measure। Outlier থাকলে MSE/RMSE বেশি sensitive।

### ৩২. R² negative হতে পারে?

হ্যাঁ। Test set-এ model-এর residual error mean-value baseline-এর চেয়েও বেশি হলে R² negative হতে পারে। R² সবসময় 0 থেকে 1-এর মধ্যে থাকবে এমন নয়।

### ৩৩. Threshold tuning-এর জন্য test set ব্যবহার করা যাবে?

না। Threshold validation data বা cross-validation-এর out-of-fold prediction দিয়ে select করবে। Test set final model ও decision freeze করার পরে একবার evaluate করবে।

### ৩৪. Calibration কখন গুরুত্বপূর্ণ?

Probability-কে risk estimate বা decision cost-এ ব্যবহার করলে calibration গুরুত্বপূর্ণ। Model ranking ভালো হলেও 0.8 score পাওয়া sample-এর 80% positive নাও হতে পারে। Reliability diagram ও Brier score দিয়ে evaluate করা যায়।

### ৩৫. Group split কখন দরকার?

একই customer, patient, device বা household-এর multiple correlated record থাকলে group split দরকার হতে পারে। একই group train ও test-এ থাকলে model entity-specific pattern memorize করে optimistic score পেতে পারে।

### ৩৬. Missing Completely at Random, MAR এবং MNAR কী?

MCAR-এ missingness observed/unobserved value-এর সঙ্গে systematic relation নেই। MAR observed variable-এর উপর নির্ভর করে। MNAR missing value বা unobserved factor-এর উপর নির্ভর করে। বাস্তবে pattern identify করা কঠিন, domain context দরকার।

### ৩৭. Data drift ও concept drift-এর পার্থক্য কী?

Data drift হলো input feature distribution বদলানো। Concept drift হলো feature ও target-এর relationship বদলানো। Drift detect করলেই retrain নয়; data pipeline, seasonality ও business change investigate করতে হবে।

### ৩৮. LEFT JOIN-এর পরে WHERE clause result কীভাবে বদলাতে পারে?

LEFT JOIN unmatched left row-কে right-side NULL দিয়ে রাখে। এরপর WHERE right_table.column = value দিলে NULL row filter out হয়; result INNER JOIN-এর মতো আচরণ করতে পারে। Right-side matching condition ON clause-এ রাখা দরকার কি না বিবেচনা করো।

### ৩৯. EXISTS ও JOIN-এর পার্থক্য কী?

EXISTS matching row আছে কি না পরীক্ষা করে এবং left row-কে duplicate করে না। JOIN matching rows combine করে; right side-এ multiple match থাকলে output row বাড়তে পারে। Use case অনুযায়ী বেছে নাও।

### ৪০. Window function ও GROUP BY-এর পার্থক্য কী?

GROUP BY group-প্রতি row produce করে, অর্থাৎ original rows collapse করে। Window function original row preserve করে এবং related rows-এর aggregate/rank পাশে যোগ করে।

### ৪১. ROW_NUMBER, RANK এবং DENSE_RANK কীভাবে আলাদা?

ROW_NUMBER সব row-কে unique sequence দেয়। RANK tie হলে একই rank দেয় এবং পরের rank-এ gap রাখে। DENSE_RANK tie-তে একই rank দেয় কিন্তু gap রাখে না।

### ৪২. Composite index-এর column order গুরুত্বপূর্ণ কেন?

Composite index-এর leading column query predicate-এর সঙ্গে match করা গুরুত্বপূর্ণ। (customer_id, created_at) index customer-specific date query-তে উপকারী হতে পারে; শুধু created_at filter-এ database dialect/planner অনুযায়ী একই benefit নাও পাওয়া যায়।

### ৪৩. EXPLAIN এবং EXPLAIN ANALYZE-এর পার্থক্য কী?

EXPLAIN estimated plan দেখায়। EXPLAIN ANALYZE query execute করে actual timing ও row count দেয়। Query optimize করার সময় estimated বনাম actual row count mismatch ও scan/join type inspect করো।

### ৪৪. SQL transaction rollback কেন দরকার?

একাধিক dependent update-কে atomic unit হিসেবে চালাতে transaction ব্যবহার হয়। মাঝখানে failure হলে rollback করে partial state এড়ানো যায়। ACID isolation level concurrent transaction-এর visibility ও anomaly-তে প্রভাব ফেলে।

### ৪৫. Offset ও keyset pagination-এর পার্থক্য কী?

Offset pagination LIMIT/OFFSET দিয়ে আগের row skip করে; বড় offset ব্যয়বহুল হতে পারে। Keyset pagination last-seen ordered key দিয়ে পরের page আনে; বড় dataset-এ efficient হতে পারে, তবে stable unique ordering দরকার।

## Evaluation and SQL Reasoning Flow

~~~mermaid
flowchart TD
    A["Business question"] --> B["Choose metric / query result"]
    B --> C["Validate data and assumptions"]
    C --> D["Check leakage / NULL / duplicate keys"]
    D --> E["Run evaluation or query"]
    E --> F["Inspect errors and execution plan"]
    F --> G["Explain result and limitation"]
~~~

## Oral Practice

প্রতিটি প্রশ্নের উত্তর definition দিয়ে শুরু করো, তারপর formula বা query behavior, practical example, edge case এবং trade-off explain করো। শুধু syntax নয়, কেন ওই approach সঠিক সেটি justify করো।
