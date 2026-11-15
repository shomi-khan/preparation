# Model Evaluation

Model train করা মানেই কাজ শেষ নয়। একটি model কতবার সঠিক predict করেছে তার চেয়েও গুরুত্বপূর্ণ হলো: **কোন ধরনের ভুল করছে, unseen data-তে কেমন করছে এবং সেই ভুলের business cost কী?** Model evaluation এই প্রশ্নগুলোর উত্তর দেয়।

## ১. Evaluation কেন দরকার?

Training data-তে ভালো score যথেষ্ট নয়। Model unseen data-তে generalize করে কি না এবং ভুল prediction-এর business impact কী, তা মাপতে evaluation দরকার।

~~~mermaid
flowchart TD
    A["Trained model"] --> B["Unseen evaluation data"]
    B --> C["Predictions"]
    C --> D["Compare with ground truth"]
    D --> E["Metrics and error analysis"]
    E --> F["Model / business decision"]
~~~

## ২. Confusion Matrix কী?

Binary classification-এ চারটি outcome থাকে:

| | Actual positive | Actual negative |
|---|---|---|
| Predicted positive | TP | FP |
| Predicted negative | FN | TN |

- **TP:** positive সঠিকভাবে positive predict।
- **TN:** negative সঠিকভাবে negative predict।
- **FP:** negative-কে ভুল positive predict।
- **FN:** positive-কে ভুল negative predict।

Fraud detection-এ FN মানে fraud transaction miss হওয়া; FP মানে legitimate transaction ভুলভাবে block হওয়া।

~~~mermaid
flowchart TD
    A["Prediction"] --> B{"Actual label"}
    B -->|Positive| C{"Predicted label"}
    B -->|Negative| D{"Predicted label"}
    C -->|Positive| TP["True Positive"]
    C -->|Negative| FN["False Negative"]
    D -->|Positive| FP["False Positive"]
    D -->|Negative| TN["True Negative"]
~~~

## ৩. Accuracy কী এবং কখন misleading?

Accuracy = (TP + TN) / (TP + TN + FP + FN)

সব prediction-এর মধ্যে কতগুলো correct তা মাপে। Class balance থাকলে useful হতে পারে, কিন্তু severe class imbalance-এ misleading।

উদাহরণ: ১০০০ transaction-এর ৯৯০টি legitimate, ১০টি fraud। সবকিছু legitimate predict করলেও accuracy 99%, কিন্তু fraud recall 0%।

## ৪. Precision কী?

Precision = TP / (TP + FP)

Positive prediction-এর মধ্যে কতটি সত্য positive। False positive-এর cost বেশি হলে গুরুত্বপূর্ণ, যেমন legitimate payment block করলে customer friction তৈরি হয়।

যদি model ১০টি transaction fraud বলে এবং ৮টি সত্যিই fraud হয়, precision = 0.8।

## ৫. Recall কী?

Recall = TP / (TP + FN)

Actual positive-এর মধ্যে কতটি শনাক্ত হয়েছে। False negative-এর cost বেশি হলে গুরুত্বপূর্ণ, যেমন fraud miss করা।

যদি ১০টি fraud-এর মধ্যে ৯টি শনাক্ত হয়, recall = 0.9।

## ৬. F1 Score কী?

F1 হলো precision ও recall-এর harmonic mean:

~~~text
F1 = 2 × (Precision × Recall) / (Precision + Recall)
~~~

দুই metric-এর balance দেখতে সাহায্য করে। তবে F1 business cost-এর পূর্ণ বিকল্প নয় এবং true negative সরাসরি বিবেচনা করে না।

## ৭. Specificity কী?

Specificity = TN / (TN + FP)

Actual negative-এর মধ্যে কতটি সঠিকভাবে negative শনাক্ত হয়েছে। Medical screening বা fraud review-তে recall-এর পাশাপাশি specificity-ও গুরুত্বপূর্ণ হতে পারে।

## ৮. ROC Curve এবং ROC-AUC কী?

বিভিন্ন threshold-এ True Positive Rate (Recall) বনাম False Positive Rate plot করলে ROC curve হয়।

- TPR = TP / (TP + FN)
- FPR = FP / (FP + TN)

ROC-AUC score ranking ability-এর summary। Random ranking-এর AUC প্রায় 0.5, perfect ranking-এর 1.0।

~~~mermaid
flowchart TD
    A["Predicted scores"] --> B["Choose many thresholds"]
    B --> C["Calculate TPR and FPR"]
    C --> D["Plot ROC curve"]
    D --> E["Area under curve: ROC-AUC"]
~~~

ROC-AUC একটি threshold-specific metric নয়। Highly imbalanced data-তে negative class অনেক বেশি থাকায় ROC-AUC optimistic-looking হতে পারে; PR curve-ও দেখো।

## ৯. Precision-Recall Curve এবং PR-AUC কী?

বিভিন্ন threshold-এ precision বনাম recall plot করলে PR curve হয়। Positive class rare হলে precision/recall trade-off অনেক সময় ROC-এর চেয়ে বেশি actionable।

Scikit-learn-এর `average_precision_score` average precision হিসাব করে; এটি সব implementation-এর trapezoidal PR-AUC-এর সঙ্গে হুবহু একই নয়। রিপোর্টে কোন metric ব্যবহার করেছ তা স্পষ্ট করো।

## ১০. Threshold Tuning কী?

Classifier probability/score দেয়। `predict()` সাধারণত default threshold ব্যবহার করে class label দেয়। Threshold বদলালে precision/recall trade-off বদলায়।

~~~mermaid
flowchart TD
    A["Predicted probability"] --> B{"Compare to threshold"}
    B -->|>= threshold| C["Positive prediction"]
    B -->|< threshold| D["Negative prediction"]
    E["Choose threshold using validation data"] --> B
~~~

Business objective অনুযায়ী validation data বা cross-validation-এর out-of-fold prediction দিয়ে threshold বেছে নাও। Test set দিয়ে threshold tune করবে না।

## ১১. Regression Model কীভাবে evaluate করব?

Classification ছাড়াও regression task-এর জন্য আলাদা metric দরকার।

| Metric | Meaning | Sensitivity |
|---|---|---|
| MAE | Absolute error-এর average | Outlier-এর প্রতি তুলনামূলক কম sensitive |
| MSE | Squared error-এর average | বড় error-কে বেশি penalty |
| RMSE | MSE-এর square root | Target-এর unit-এ interpretable |
| R² | Mean baseline-এর তুলনায় variance explain করার measure | Negative-ও হতে পারে |

~~~python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np

mae = mean_absolute_error(y_test, predictions)
rmse = np.sqrt(mean_squared_error(y_test, predictions))
r2 = r2_score(y_test, predictions)
~~~

R² 1 মানে perfect fit, 0 মানে test set-এ mean predictor-এর সমতুল্য score, negative মানে mean baseline-এর চেয়েও খারাপ হতে পারে।

## ১২. Cross-validation কী?

K-fold CV-তে data K ভাগ হয়; প্রতিবার একটি fold validation এবং বাকিগুলো training। সব fold-এর score summarize করা হয়।

~~~mermaid
flowchart TD
    A["Dataset"] --> B["Fold 1"]
    A --> C["Fold 2"]
    A --> D["Fold 3"]
    B --> E["Train on other folds"]
    C --> E
    D --> E
    E --> F["Collect validation scores"]
    F --> G["Mean and variability"]
~~~

Classification-এ StratifiedKFold class proportion ধরে রাখে। Grouped data-তে GroupKFold, time series-এ temporal split দরকার হতে পারে। CV training/model selection-এর জন্য; final test untouched রাখো।

## ১৩. Imbalanced Dataset কীভাবে evaluate করব?

সমাধান/কৌশল:
- Class weight।
- Threshold tuning।
- Oversampling/undersampling।
- Appropriate metrics।
- More representative data।

Resampling অবশ্যই training fold-এর ভেতরে হবে, validation/test-এ নয়। Cross-validation-এ resampling step Pipeline-এর মধ্যে রাখো (প্রয়োজনে imbalanced-learn Pipeline)।

## ১৪. Calibration কী?

Model probability 0.8 দিলে, এমন prediction-গুলোর প্রায় ৮০% positive হওয়া calibration-এর intuitive অর্থ। Ranking ভালো হলেই probability calibrated হবে এমন নয়।

Reliability diagram ও Brier score কাজে লাগে। Calibration দরকার কি না তা নির্ভর করে probability দিয়ে business decision নেওয়া হবে কি না।

## ১৫. Confidence Interval এবং Metric Variability কেন গুরুত্বপূর্ণ?

একটি test score sample-এর উপর নির্ভর করে। ছোট test set-এ metric estimate uncertain হতে পারে। Cross-validation fold score, bootstrap confidence interval বা repeated evaluation uncertainty সম্পর্কে ধারণা দিতে পারে।

একটি model 0.82 এবং অন্যটি 0.83 score করলে, uncertainty বিবেচনা ছাড়া দ্বিতীয়টিকে নিশ্চিতভাবে ভালো বলা ঠিক নয়।

## ১৬. Scikit-learn Example

~~~python
from sklearn.metrics import (
    confusion_matrix,
    precision_score,
    recall_score,
    f1_score,
    roc_auc_score,
    average_precision_score,
)

pred = model.predict(X_test)
proba = model.predict_proba(X_test)[:, 1]

print(confusion_matrix(y_test, pred))
print("precision", precision_score(y_test, pred, zero_division=0))
print("recall", recall_score(y_test, pred, zero_division=0))
print("f1", f1_score(y_test, pred, zero_division=0))
print("roc_auc", roc_auc_score(y_test, proba))
print("average_precision", average_precision_score(y_test, proba))
~~~

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Accuracy? | Correct prediction / total |
| Precision? | TP / (TP + FP) |
| Recall? | TP / (TP + FN) |
| Specificity? | TN / (TN + FP) |
| F1? | Precision ও recall-এর harmonic mean |
| ROC-AUC? | TPR বনাম FPR-এর threshold sweep summary |
| PR-AUC? | Precision বনাম recall trade-off summary |
| Threshold কোথায় tune? | Validation/CV prediction-এ |
| MAE? | Average absolute regression error |
| RMSE? | Square root of mean squared error |
| R² negative হতে পারে? | হ্যাঁ |
| Calibration? | Predicted probability বনাম observed frequency alignment |
| CV? | Multiple fold-এ train/validate |
| Resampling কোথায়? | শুধু training fold-এ |
| Metric variability? | Estimate-এর uncertainty/stability |

## Practice

একটি imbalanced classification dataset-এ:
1. Confusion matrix তৈরি করো।
2. Accuracy, precision, recall, specificity ও F1 বের করো।
3. ROC curve এবং PR curve plot করো।
4. ROC-AUC ও average precision compare করো।
5. Validation data-তে threshold tune করো।
6. Class weight বা training-only resampling experiment করো।
7. Regression dataset-এ MAE, RMSE ও R² compare করো।
8. Cross-validation score-এর mean ও standard deviation report করো।
9. Calibration curve ও Brier score inspect করো।
10. কোন metric business decision-এর সঙ্গে সবচেয়ে relevant, তা justify করো।

## Practice করার পদ্ধতি

1. Metric বেছে নেওয়ার আগে false positive ও false negative-এর cost লিখে নাও।
2. Probability-based metric এবং threshold-based metric আলাদা করে বুঝো।
3. Test set final evaluation-এর আগে threshold বা model tune কোরো না।
4. Class imbalance, sample size ও metric uncertainty বিবেচনা করো।
5. Score-এর পাশাপাশি confusion matrix ও error examples analyze করো।
