# Model Evaluation

## ১. Evaluation কেন দরকার?
Training data-তে ভালো score যথেষ্ট নয়। Model unseen data-তে কেমন generalize করে এবং ভুল prediction-এর business impact কী, তা মাপতে evaluation দরকার।

## ২. Confusion Matrix
Binary classification-এ:
- TP: positive সঠিকভাবে positive predict।
- TN: negative সঠিকভাবে negative predict।
- FP: negative-কে ভুল positive predict।
- FN: positive-কে ভুল negative predict।

Fraud detection-এ FN মানে fraud transaction miss হওয়া; FP মানে legitimate transaction ভুলভাবে block হওয়া।

## ৩. Accuracy
`(TP + TN) / (TP + TN + FP + FN)`। Class imbalance থাকলে misleading হতে পারে।

## ৪. Precision
`TP / (TP + FP)`। Positive prediction-এর মধ্যে কতটি সত্য positive। False positive-এর cost বেশি হলে গুরুত্বপূর্ণ।

## ৫. Recall
`TP / (TP + FN)`। Actual positive-এর মধ্যে কতটি শনাক্ত হয়েছে। False negative-এর cost বেশি হলে গুরুত্বপূর্ণ।

## ৬. F1 Score
Precision ও recall-এর harmonic mean: `2PR/(P+R)`। দুইটির balance দেখতে সাহায্য করে; business cost-এর পূর্ণ বিকল্প নয়।

## ৭. Specificity
`TN / (TN + FP)`। Actual negative-এর মধ্যে কতটি সঠিকভাবে negative শনাক্ত হয়েছে।

## ৮. ROC Curve ও AUC
বিভিন্ন threshold-এ True Positive Rate বনাম False Positive Rate plot করে ROC curve। ROC-AUC ranking ability-এর একটি summary; severe imbalance-এ Precision-Recall curve-ও দেখা জরুরি।

## ৯. PR-AUC
Precision-Recall curve-এর area। Positive class rare হলে অনেক সময় ROC-AUC-এর চেয়ে বেশি informative।

## ১০. Threshold Tuning
`predict()` সাধারণত default threshold ব্যবহার করে। Business objective অনুযায়ী probability threshold বদলালে precision/recall বদলায়। Threshold validation data-তে বেছে test set-এ একবার evaluate করবে।

## ১১. Cross-validation
K-fold CV-তে data K ভাগ হয়; প্রতিবার একটি fold validation এবং বাকিগুলো training। Classification-এ StratifiedKFold class proportion ধরে রাখে। Time series-এ random K-fold উপযুক্ত নয়; temporal split দরকার।

## ১২. Imbalanced Dataset
সমাধান:
- Class weight
- Threshold tuning
- Oversampling/undersampling
- Appropriate metrics
- More representative data
Resampling অবশ্যই training fold-এর ভেতরে হবে, validation/test-এ নয়।

## ১৩. Calibration
যে prediction-এ probability 0.8 বলা হচ্ছে, অনুরূপ prediction-এর প্রায় ৮০% সত্য হওয়া calibration-এর লক্ষ্য। Reliability diagram ও Brier score কাজে লাগে।

## ১৪. Scikit-learn Example
```python
from sklearn.metrics import (
    confusion_matrix, precision_score, recall_score,
    f1_score, roc_auc_score, average_precision_score
)

pred = model.predict(X_test)
proba = model.predict_proba(X_test)[:, 1]

print(confusion_matrix(y_test, pred))
print("precision", precision_score(y_test, pred, zero_division=0))
print("recall", recall_score(y_test, pred, zero_division=0))
print("f1", f1_score(y_test, pred, zero_division=0))
print("roc_auc", roc_auc_score(y_test, proba))
print("pr_auc", average_precision_score(y_test, proba))
```

## Practice
একটি imbalanced dataset-এ accuracy, precision, recall, F1, ROC-AUC ও PR-AUC compare করো। False positive ও false negative-এর business cost ব্যাখ্যা করো।
