# Machine Learning Algorithms

একই dataset-এ একাধিক algorithm ভিন্ন ধরনের pattern শিখতে পারে। কোনো algorithm সব problem-এ best নয়। Model বেছে নেওয়ার আগে target, data size, feature type, interpretability, latency এবং evaluation metric বুঝতে হয়।

এই chapter-এ common supervised ও unsupervised algorithm-এর intuition, assumptions, strengths এবং limitations আলোচনা করা হয়েছে।

## ১. Linear Regression কীভাবে কাজ করে?

Linear Regression continuous numeric target predict করতে input feature-এর weighted linear combination শেখে।

~~~text
prediction = w1*x1 + w2*x2 + ... + b
~~~

অর্থাৎ model এমন coefficient শেখে যাতে prediction ও actual value-এর error কম হয়।

~~~mermaid
flowchart LR
    A["Input features X"] --> B["Weighted sum: Xw + b"]
    B --> C["Predicted value"]
    C --> D["Compare with actual y"]
    D --> E["Optimize coefficients"]
~~~

**ব্যবহার:** baseline regression, trend estimation, interpretable relationship।

**Limitation:** linear relationship assumption; outlier, multicollinearity ও nonlinear pattern প্রভাব ফেলতে পারে। Feature transformation বা interaction term লাগতে পারে।

## ২. Logistic Regression কেন classification algorithm?

নামের মধ্যে Regression থাকলেও Logistic Regression classification-এ ব্যবহৃত হয়। এটি feature-এর linear combination-কে sigmoid function দিয়ে probability-তে map করে।

~~~python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)

probabilities = model.predict_proba(X_test)
predictions = model.predict(X_test)
~~~

~~~mermaid
flowchart TD
    A["Features X"] --> B["Linear score: Xw + b"]
    B --> C["Sigmoid / probability"]
    C --> D["Threshold"]
    D --> E["Predicted class"]
~~~

Binary classification-এ শক্তিশালী baseline। Feature scaling উপকারী হতে পারে। Default threshold 0.5 সব business problem-এর জন্য optimal নয়।

## ৩. Decision Tree কীভাবে কাজ করে?

Decision Tree feature-এর condition/threshold দিয়ে data-কে recursiveভাবে split করে। প্রতিটি leaf node prediction দেয়।

~~~mermaid
flowchart TD
    A["All samples"] --> B{"Amount > 1000?"}
    B -->|Yes| C{"New device?"}
    B -->|No| D["Low risk"]
    C -->|Yes| E["High risk"]
    C -->|No| F["Medium risk"]
~~~

**Strengths**
- Explain করা তুলনামূলক সহজ।
- Nonlinear relationship ও feature interaction ধরতে পারে।
- Numerical feature scaling সাধারণত দরকার হয় না।

**Limitations**
- গভীর tree overfit করতে পারে।
- Data-র ছোট পরিবর্তনে tree structure বদলে যেতে পারে।
- একক tree-এর prediction boundary অস্থির হতে পারে।

## ৪. Random Forest কীভাবে কাজ করে?

Random Forest অনেক Decision Tree train করে এবং তাদের prediction combine করে। Classification-এ voting, regression-এ averaging ব্যবহার হয়।

প্রতিটি tree সাধারণত bootstrap sample এবং feature-এর random subset ব্যবহার করে। এতে tree-গুলোর correlation কমে এবং variance কমাতে সাহায্য করে।

~~~mermaid
flowchart TD
    A["Training data"] --> B["Bootstrap sample 1"]
    A --> C["Bootstrap sample 2"]
    A --> D["Bootstrap sample N"]
    B --> E["Tree 1"]
    C --> F["Tree 2"]
    D --> G["Tree N"]
    E --> H["Aggregate predictions"]
    F --> H
    G --> H
    H --> I["Final prediction"]
~~~

~~~python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=200,
    random_state=42,
    class_weight="balanced",
)
~~~

**Trade-off:** Nonlinear tabular data-তে ভালো starting point হতে পারে; একক tree-এর তুলনায় interpretability কম এবং model বড় হতে পারে।

## ৫. K-Nearest Neighbors (KNN) কীভাবে কাজ করে?

KNN নতুন sample-এর সবচেয়ে কাছের Kটি training sample খুঁজে prediction করে। Classification-এ majority vote, regression-এ neighbor value average হতে পারে।

~~~mermaid
flowchart TD
    A["New sample"] --> B["Calculate distance to training points"]
    B --> C["Select K nearest neighbors"]
    C --> D["Vote / average"]
    D --> E["Prediction"]
~~~

**গুরুত্বপূর্ণ বিষয়**
- Distance metric নির্বাচন গুরুত্বপূর্ণ।
- Feature scaling সাধারণত প্রয়োজন।
- বড় dataset-এ inference ধীর হতে পারে।
- High-dimensional space-এ distance কম informative হতে পারে।
- K ছোট হলে noise sensitivity বাড়ে; K বড় হলে decision boundary smooth হয়।

## ৬. Support Vector Machine (SVM) কী?

SVM class-গুলোর মধ্যে margin সর্বোচ্চ করার চেষ্টা করে এমন decision boundary শেখে। Kernel ব্যবহার করে nonlinear boundary model করা যায়।

~~~mermaid
flowchart TD
    A["Labeled points"] --> B["Find separating boundary"]
    B --> C["Maximize margin"]
    C --> D["Support vectors define boundary"]
    D --> E["Predict new point"]
~~~

**Strengths:** medium-sized/high-dimensional data-তে উপযোগী হতে পারে; kernel দিয়ে nonlinear separation।

**Limitations:** বড় dataset-এ training expensive হতে পারে; feature scaling দরকার; probability output সরাসরি পাওয়া যায় না (probability estimate চাইলে calibration/configuration লাগতে পারে)।

## ৭. Naive Bayes কী?

Naive Bayes Bayes' theorem ব্যবহার করে class probability estimate করে এবং feature-গুলোর conditional independence assumption নেয়।

~~~mermaid
flowchart LR
    A["Features"] --> B["Class likelihoods"]
    C["Class prior"] --> D["Bayes rule"]
    B --> D
    D --> E["Posterior class probability"]
    E --> F["Predicted class"]
~~~

Text classification ও spam filtering-এর মতো problem-এ দ্রুত baseline হতে পারে। Independence assumption বাস্তবে পুরোপুরি সত্য না হলেও model কার্যকর হতে পারে। Feature type অনুযায়ী Gaussian, Multinomial বা Bernoulli Naive Bayes বেছে নিতে হয়।

## ৮. K-Means Clustering কীভাবে কাজ করে?

K-Means unlabeled data-কে Kটি cluster-এ ভাগ করে centroid-এর distance-based objective কমানোর চেষ্টা করে।

~~~mermaid
flowchart TD
    A["Choose K initial centroids"] --> B["Assign each point to nearest centroid"]
    B --> C["Recalculate centroids"]
    C --> D{"Assignments / centroids stable?"}
    D -->|না| B
    D -->|হ্যাঁ| E["Final clusters"]
~~~

~~~python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, random_state=42, n_init="auto")
cluster_ids = model.fit_predict(X_scaled)
~~~

**Limitations:** K আগে নির্ধারণ করতে হয়; feature scaling গুরুত্বপূর্ণ; outlier ও non-spherical cluster-এ সমস্যা হতে পারে। Cluster label-এর numeric value নিজে থেকে কোনো semantic rank নয়।

## ৯. Gradient Boosting কীভাবে কাজ করে?

Boosting sequentially weak learners, সাধারণত shallow trees, যোগ করে আগের ensemble-এর error কমানোর চেষ্টা করে। প্রতিটি নতুন learner আগের model-এর residual বা loss gradient-এর উপর কাজ করে।

~~~mermaid
flowchart TD
    A["Initial prediction"] --> B["Calculate errors / gradients"]
    B --> C["Fit next weak learner"]
    C --> D["Add learner to ensemble"]
    D --> E{"More boosting rounds?"}
    E -->|হ্যাঁ| B
    E -->|না| F["Final ensemble"]
~~~

XGBoost, LightGBM ও CatBoost gradient boosting family-এর popular implementation।

**Trade-off:** Tabular data-তে শক্তিশালী হতে পারে, তবে hyperparameter tuning, compute এবং overfitting control দরকার। Feature importance-কে causal explanation ধরে নেওয়া যাবে না।

## ১০. Bagging এবং Boosting-এর পার্থক্য কী?

- **Bagging:** আলাদা sample-এ একাধিক learner train করে prediction aggregate করে; variance কমাতে সাহায্য করে।
- **Boosting:** learner sequentially যোগ করে আগের error কমাতে শেখে; bias কমাতে পারে, তবে tuning-sensitive হতে পারে।

~~~mermaid
flowchart TD
    A["Ensemble methods"] --> B["Bagging"]
    A --> C["Boosting"]
    B --> D["Parallel / independent learners"]
    D --> E["Aggregate predictions"]
    C --> F["Sequential learners"]
    F --> G["Correct previous errors"]
    G --> H["Combined prediction"]
~~~

Random Forest bagging-এর উদাহরণ; Gradient Boosting boosting-এর উদাহরণ।

## ১১. কোন algorithm কখন starting point হতে পারে?

| পরিস্থিতি | সম্ভাব্য starting point |
|---|---|
| Interpretable numeric prediction | Linear Regression |
| Interpretable binary classification baseline | Logistic Regression |
| Simple rule-like nonlinear model | Decision Tree |
| Nonlinear tabular data | Random Forest |
| Strong tabular performance target | Gradient Boosting / XGBoost family |
| Local similarity meaningful, modest dataset | KNN |
| Margin-based classification | SVM |
| Text count features-এর quick baseline | Multinomial Naive Bayes |
| Unlabeled segmentation | K-Means |

কোনো algorithm সর্বক্ষেত্রে সেরা নয়। Baseline, validation, domain knowledge এবং operational constraint দিয়ে সিদ্ধান্ত নিতে হবে।

## ১২. Feature Scaling কেন এবং কোন algorithm-এ দরকার?

অনেক algorithm distance বা gradient-এর উপর নির্ভর করে, তাই feature scale গুরুত্বপূর্ণ।

- StandardScaler: training data-এর mean 0 ও standard deviation 1 করার চেষ্টা।
- MinMaxScaler: নির্দিষ্ট range-এ scale।
- Tree-based model সাধারণত scaling-এর উপর কম নির্ভরশীল।

~~~python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
~~~

~~~mermaid
flowchart TD
    A["Split data first"] --> B["Fit scaler on X_train"]
    B --> C["Transform X_train"]
    B --> D["Transform X_test using same scaler"]
~~~

**Critical rule:** Scaler শুধু training data-তে fit করবে। Test data-তে শুধু transform। Cross-validation-এ preprocessing ও model-কে Pipeline-এ রাখো, যাতে প্রতিটি fold-এ leakage না হয়।

## ১৩. Regularization কী?

Model-এর complexity নিয়ন্ত্রণ করতে objective/loss function-এ penalty যোগ করা হয়।

- **L1 (Lasso):** কিছু coefficient zero করতে পারে; feature selection-এর মতো effect দিতে পারে।
- **L2 (Ridge):** coefficient ছোট করে, সাধারণত zero করে না।
- Logistic Regression-এও regularization ব্যবহৃত হয়।

~~~mermaid
flowchart LR
    A["Training objective"] --> B["Prediction loss"]
    A --> C["Complexity penalty"]
    B --> D["Loss + penalty"]
    C --> D
    D --> E["Optimized parameters"]
~~~

Regularization strength খুব বেশি হলে underfitting, খুব কম হলে overfitting হতে পারে। Hyperparameter validation data/cross-validation দিয়ে নির্বাচন করো।

## ১৪. Feature importance কি model explanation বা causation প্রমাণ করে?

না। Tree-based feature importance বা permutation importance model prediction-এ feature-এর ভূমিকা সম্পর্কে ধারণা দেয়, কিন্তু causal relationship প্রমাণ করে না। Correlated feature থাকলে importance ভাগ হয়ে যেতে পারে বা misleading হতে পারে। Explanation দরকার হলে permutation importance, SHAP এবং domain analysis বিবেচনা করো।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Linear Regression? | Continuous target-এর linear prediction |
| Logistic Regression? | Classification-এর probability model |
| Decision Tree? | Feature condition দিয়ে recursive split |
| Random Forest? | Bootstrap/random feature-সহ many trees aggregate |
| KNN? | Nearest samples থেকে prediction |
| SVM? | Margin maximize করা boundary |
| Naive Bayes? | Bayes rule + conditional independence assumption |
| K-Means? | Centroid-based clustering |
| Gradient Boosting? | Sequential weak learners দিয়ে error reduce |
| Bagging? | Independent learners aggregate |
| Boosting? | Sequential learners improve ensemble |
| Scaling কার দরকার? | KNN/SVM/linear gradient models-এ সাধারণত; tree-তে কম |
| L1? | Coefficient zero করতে পারে |
| L2? | Coefficient shrink করে |
| Feature importance causal? | না |

## Interview Practice

1. Logistic Regression কেন classification algorithm?
2. Decision Tree overfit করে কেন?
3. Random Forest কীভাবে variance কমায়?
4. KNN-এ scaling কেন গুরুত্বপূর্ণ?
5. SVM margin বলতে কী বোঝায়?
6. Naive Bayes-এর independence assumption কী?
7. K-Means-এর limitation কী?
8. Boosting ও bagging-এর high-level পার্থক্য কী?
9. Gradient Boosting sequential কেন?
10. Linear model-এর regularization কেন দরকার?
11. Scaling train data-তে fit করে test data-তে transform করতে হয় কেন?
12. Feature importance কি causation প্রমাণ করে?

## Practice করার পদ্ধতি

1. Algorithm নির্বাচন করার আগে target ও data shape বুঝে নাও।
2. Simple baseline তৈরি করে candidate model-এর সঙ্গে compare করো।
3. Scaling, imputation ও encoding Pipeline-এর মধ্যে রাখো।
4. Model complexity, inference latency এবং interpretability বিবেচনা করো।
5. শুধু score নয়, error pattern ও failure mode analyze করো।
