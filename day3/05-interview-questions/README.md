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
