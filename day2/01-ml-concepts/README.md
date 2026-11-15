# Machine Learning Concepts

Machine Learning বুঝতে হলে আগে একটি প্রশ্ন করতে হয়: আমরা কি computer-কে প্রতিটি পরিস্থিতির জন্য rule লিখে দিচ্ছি, নাকি example দেখিয়ে এমন একটি model তৈরি করছি যা নতুন পরিস্থিতিতে prediction করতে পারে?

এই chapter-এ ML-এর core concepts, training process এবং real-world failure mode আলোচনা করা হয়েছে। এখানে মূল লক্ষ্য algorithm মুখস্থ করা নয়; problem framing, data split, generalization ও leakage বুঝতে পারা।

## ১. Machine Learning কী?

Machine Learning (ML) হলো এমন পদ্ধতি যেখানে computer data থেকে pattern শেখে এবং সেই pattern ব্যবহার করে নতুন input-এর prediction বা decision নেয়। প্রতিটি case-এর জন্য rule হাতে লিখে দিতে হয় না।

উদাহরণ: আগের transaction-এর amount, merchant, time ও device-related feature দেখে নতুন transaction fraud হওয়ার সম্ভাবনা অনুমান করা।

~~~mermaid
flowchart TD
    A["Historical data"] --> B["Learning algorithm"]
    B --> C["Trained model"]
    D["New unseen input"] --> E["Trained model"]
    C --> E
    E --> F["Prediction"]
~~~

## ২. Traditional Programming এবং Machine Learning-এর পার্থক্য কী?

Traditional programming-এ developer input ও explicit rules দেয়, program output তৈরি করে। ML training-এ input-এর সঙ্গে example answer/label দিয়ে algorithm model parameters শেখে।

~~~mermaid
flowchart TD
    A["Traditional programming"] --> B["Input + Hand-written rules"]
    B --> C["Output"]
    D["ML training"] --> E["Input + Expected labels"]
    E --> F["Learning algorithm"]
    F --> G["Learned model"]
    H["ML inference: New input + model"] --> I["Prediction"]
~~~

ML মানে rules একেবারে নেই, তা নয়। Data validation, feature transformation, decision threshold এবং business policy-তে এখনও explicit rules থাকে।

## ৩. Supervised Learning কী?

যে training dataset-এ input-এর পাশাপাশি expected answer বা label থাকে, সেটি supervised learning। Model input থেকে label predict করার mapping শেখে।

দুটি প্রধান task:

- **Classification:** category predict করা, যেমন fraud / legitimate।
- **Regression:** continuous numeric value predict করা, যেমন house price।

~~~mermaid
flowchart TD
    A["Labeled examples (X, y)"] --> B["Supervised learning"]
    B --> C{"Target type?"}
    C -->|Category| D["Classification"]
    C -->|Continuous number| E["Regression"]
~~~

## ৪. Unsupervised Learning কী?

যে data-তে explicit target label নেই, সেখানে structure বা pattern খোঁজার পদ্ধতি হলো unsupervised learning।

উদাহরণ:
- Clustering: similar customer group করা।
- Dimensionality reduction: কম dimension-এ data represent করা।
- Anomaly exploration: unusual observation খুঁজে দেখা।

~~~mermaid
flowchart TD
    A["Unlabeled data X"] --> B["Unsupervised method"]
    B --> C["Clusters"]
    B --> D["Lower-dimensional representation"]
    B --> E["Structure / unusual observations"]
~~~

Unsupervised model-এর output সবসময় business-meaningful category হবে এমন নয়। Cluster-কে domain context দিয়ে interpret করতে হয়।

## ৫. Semi-supervised, Self-supervised এবং Reinforcement Learning কী?

- **Semi-supervised learning:** অল্প labeled data ও বেশি unlabeled data একসঙ্গে ব্যবহার করে।
- **Self-supervised learning:** data থেকেই supervision signal তৈরি করে, যেমন language model-এর next-token prediction।
- **Reinforcement learning:** agent environment-এ action নেয় এবং reward/feedback থেকে policy শেখে।

~~~mermaid
flowchart TD
    A["Learning setup"] --> B["Few labels + many unlabeled"]
    A --> C["Labels derived from data itself"]
    A --> D["Actions + rewards from environment"]
    B --> E["Semi-supervised"]
    C --> F["Self-supervised"]
    D --> G["Reinforcement learning"]
~~~

## ৬. Classification এবং Regression-এর পার্থক্য কী?

| বিষয় | Classification | Regression |
|---|---|---|
| Output | Class/category | Continuous number |
| উদাহরণ | Spam / Not spam | আগামী মাসের sales |
| Common metric | Precision, Recall, F1, ROC-AUC | MAE, MSE, RMSE, R² |
| Example model | Logistic Regression, Decision Tree Classifier | Linear Regression, Random Forest Regressor |

একই feature ব্যবহার করলেও target-এর nature অনুযায়ী problem type বদলায়।

## ৭. Feature এবং Label কী?

- **Feature (X):** model-এর input variable।
- **Label/Target (y):** model যে outcome predict করতে শেখে।

Fraud detection-এ amount, merchant category, time, device age feature হতে পারে; fraud status label হতে পারে।

~~~mermaid
flowchart LR
    A["Transaction record"] --> B["Features X<br/>amount, merchant, time"]
    A --> C["Label y<br/>fraud / legitimate"]
    B --> D["Model learns mapping"]
    C --> D
~~~

Feature যেন prediction করার সময় বাস্তবে available থাকে। ভবিষ্যতের তথ্য feature-এ ঢুকে গেলে leakage হতে পারে।

## ৮. Training, Validation এবং Test Set কেন আলাদা?

- **Training set:** model parameters শেখে।
- **Validation set:** model/hyperparameter নির্বাচন ও tuning-এ ব্যবহৃত হয়।
- **Test set:** final performance estimate-এর জন্য একেবারে শেষে ব্যবহৃত হয়।

~~~mermaid
flowchart TD
    A["Full dataset"] --> B["Training set"]
    A --> C["Validation set"]
    A --> D["Test set"]
    B --> E["Fit model parameters"]
    C --> F["Choose model / hyperparameters"]
    E --> F
    F --> G["Freeze final model"]
    G --> H["Evaluate once on test set"]
~~~

Test set বারবার দেখে model tune করলে সেটিও effectively training process-এর অংশ হয়ে যায় এবং final estimate optimistic হতে পারে।

## ৯. Train/Test Split কীভাবে করব?

Scikit-learn-এর `train_test_split` দিয়ে split করা যায়।

~~~python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)
~~~

- `test_size=0.2`: ২০% test data।
- `random_state`: reproducible split।
- `stratify=y`: classification-এ class proportion কাছাকাছি রাখে।

**কিন্তু random split সব problem-এর জন্য সঠিক নয়।** Time-series data-তে future data train-এ ঢুকে গেলে leakage হয়। একই user/patient-এর record একাধিক split-এ গেলে group leakage হতে পারে। Data generation process অনুযায়ী split strategy বেছে নাও।

## ১০. Cross-validation কী?

একটি train/validation split-এর উপর নির্ভর না করে training data-কে একাধিক fold-এ ভাগ করে model কয়েকবার train/evaluate করাকে cross-validation বলে। এতে performance estimate split-এর উপর কতটা sensitive তা বোঝা যায়।

~~~mermaid
flowchart TD
    A["Training data"] --> B["Fold 1 validation"]
    A --> C["Fold 2 validation"]
    A --> D["Fold 3 validation"]
    B --> E["Train on remaining folds"]
    C --> E
    D --> E
    E --> F["Average / inspect fold scores"]
~~~

Classification-এ সাধারণত StratifiedKFold class proportion ধরে রাখতে পারে। Group বা time dependency থাকলে GroupKFold বা TimeSeriesSplit-এর মতো strategy দরকার হতে পারে।

## ১১. Overfitting এবং Underfitting কী?

- **Overfitting:** training data-র noise-সহ pattern শিখে ফেলে; training performance ভালো, unseen performance খারাপ।
- **Underfitting:** model যথেষ্ট pattern ধরতে পারে না; training ও validation উভয় performance খারাপ।

~~~mermaid
flowchart TD
    A["Model fit"] --> B{"Train and validation behavior"}
    B --> C["Both poor"]
    B --> D["Train good, validation poor"]
    B --> E["Both good and close"]
    C --> F["Underfitting"]
    D --> G["Overfitting"]
    E --> H["Potentially good generalization"]
~~~

সমাধান হিসেবে model complexity, regularization, feature quality, more representative data এবং cross-validation বিবেচনা করা যায়।

## ১২. Bias-Variance Trade-off কী?

- **High bias:** model অতিরিক্ত simple; systematic error বেশি।
- **High variance:** training data পরিবর্তনে model prediction অনেক বদলে যায়।

Model complexity বাড়ালে bias কমতে পারে, কিন্তু variance বাড়তে পারে। লক্ষ্য শুধু training score বাড়ানো নয়; unseen data-তে generalization।

## ১৩. Generalization কী?

Training-এর বাইরে নতুন, unseen data-তে model কতটা ভালো কাজ করে সেটিই generalization। Training accuracy খুব বেশি হলেই model ভালো নয়; validation/test performance ও real-world distribution বিবেচনা করতে হবে।

Model-এর performance production-এ কমে যেতে পারে কারণ:
- Training data representative ছিল না।
- Data distribution বদলে গেছে (data drift)।
- Label relationship বদলে গেছে (concept drift)।
- Feature pipeline training ও serving-এ আলাদা হয়েছে।

## ১৪. Feature Engineering কী?

Raw data থেকে useful input variable তৈরি বা transform করা।

উদাহরণ:
- Timestamp থেকে hour, weekday।
- Date থেকে account age।
- Transaction history থেকে rolling average।
- Text থেকে TF-IDF features।
- Category থেকে one-hot encoding।

Feature তৈরি করার সময় target leakage এবং train/test boundary মানতে হবে। Learned transformation train data-তে fit করে validation/test data-তে apply করো।

## ১৫. Data Leakage কী?

যখন training বা evaluation-এর সময় এমন তথ্য model পায় যা বাস্তব prediction time-এ পাওয়া সম্ভব নয়, অথবা test data-র information training process-এ ঢুকে যায়, তখন data leakage হয়।

উদাহরণ:
- Split করার আগে পুরো dataset-এর mean দিয়ে missing value fill করা।
- ভবিষ্যতে জানা যাবে এমন column feature হিসেবে ব্যবহার করা।
- একই user-এর correlated record train ও test উভয় জায়গায় রাখা।
- Target label-এর পরে তৈরি হওয়া field feature-এ ঢোকানো।

~~~mermaid
flowchart TD
    A["Full dataset"] --> B["Incorrect: preprocess all data"]
    B --> C["Train / test split"]
    C --> D["Test information leaked into preprocessing"]
    E["Correct: split first"] --> F["Fit preprocessing on train"]
    F --> G["Transform validation / test"]
    G --> H["Evaluate without fitting on test"]
~~~

Scikit-learn Pipeline preprocessing ও model একত্রে রাখে, যাতে cross-validation-এর প্রতিটি fold-এ transformation শুধু training fold-এ fit হয়।

## ১৬. Baseline Model কেন দরকার?

Baseline হলো simple reference model, যার সঙ্গে পরবর্তী model তুলনা করা যায়। Classification-এ DummyClassifier বা simple Logistic Regression দিয়ে শুরু করা যায়। Regression-এ mean/median predictor baseline হতে পারে।

~~~mermaid
flowchart TD
    A["Define metric"] --> B["Build simple baseline"]
    B --> C["Evaluate baseline"]
    C --> D["Train candidate model"]
    D --> E["Compare on same validation strategy"]
    E --> F{"Meaningful improvement?"}
    F -->|হ্যাঁ| G["Investigate / tune"]
    F -->|না| H["Revisit data, features, objective"]
~~~

Baseline ছাড়া complex model আসলেই উন্নতি করেছে কি না বোঝা কঠিন।

## ১৭. Loss Function এবং Evaluation Metric কি একই?

না।
- **Loss function:** training-এর সময় model parameter optimize করতে ব্যবহৃত objective।
- **Evaluation metric:** model performance বোঝাতে ব্যবহৃত measure।

কখনও loss ও metric একই family-এর হতে পারে, কিন্তু উদ্দেশ্য আলাদা। উদাহরণ: model log loss optimize করলেও business team precision/recall বা cost-sensitive metric দেখতে পারে।

## ১৮. Class Imbalance কী এবং কেন সমস্যা?

কোনো class অন্য class-এর তুলনায় অনেক বেশি থাকলে class imbalance হয়। উদাহরণ: fraud dataset-এ ১% fraud, ৯৯% legitimate।

এ অবস্থায় সব transaction-কে legitimate বললেও accuracy 99% হতে পারে, কিন্তু fraud শনাক্ত হয় না। তাই confusion matrix, precision, recall, F1, PR-AUC এবং business cost দেখতে হবে।

## ১৯. Prediction threshold কেন গুরুত্বপূর্ণ?

অনেক classifier probability বা score দেয়। Default threshold 0.5 সব business problem-এর জন্য optimal নয়। Threshold বদলালে precision ও recall-এর trade-off বদলায়।

~~~mermaid
flowchart TD
    A["Model score / probability"] --> B{"Compare with threshold"}
    B -->|Above| C["Positive prediction"]
    B -->|Below| D["Negative prediction"]
    E["Choose threshold using validation data and business cost"] --> B
~~~

Threshold test set দেখে tune কোরো না; validation data বা cross-validation-এর prediction ব্যবহার করো।

## ২০. Model Calibration কী?

Model probability 0.8 দিলে, এমন prediction-গুলোর প্রায় 80% positive হওয়া calibration-এর intuitive অর্থ। Ranking ভালো হলেই probability calibrated হবে এমন নয়। Risk score বা decision cost probability-এর উপর নির্ভর করলে calibration গুরুত্বপূর্ণ।

## ২১. Batch Learning এবং Online Learning কী?

- **Batch learning:** নির্দিষ্ট training dataset দিয়ে model train করে deploy করা হয়; update করতে পুনরায় training লাগে।
- **Online learning:** data আসার সঙ্গে সঙ্গে বা ছোট batch-এ model incrementally update হতে পারে।

~~~mermaid
flowchart TD
    A["New data"] --> B["Batch learning"]
    A --> C["Online learning"]
    B --> D["Periodic full / scheduled retraining"]
    C --> E["Incremental model updates"]
~~~

Online learning-এ data quality, drift, feedback loop ও rollback strategy বিশেষ গুরুত্বপূর্ণ।

## ২২. ML Lifecycle কী?

~~~mermaid
flowchart TD
    A["Problem definition"] --> B["Data collection"]
    B --> C["Data validation / cleaning"]
    C --> D["Feature engineering"]
    D --> E["Train / validation"]
    E --> F["Model evaluation"]
    F --> G{"Meets objective?"}
    G -->|না| D
    G -->|হ্যাঁ| H["Deployment"]
    H --> I["Monitoring"]
    I --> J["Drift / performance review"]
    J --> K["Retraining / improvement"]
    K --> E
~~~

Deployment-এর পর কাজ শেষ নয়। Prediction quality, latency, failure rate, data drift এবং business outcome monitor করতে হয়।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| ML কী? | Data থেকে pattern শিখে unseen input predict/decide করা |
| Supervised? | Labeled data দিয়ে learning |
| Unsupervised? | Explicit target ছাড়া structure খোঁজা |
| Classification? | Category predict |
| Regression? | Continuous value predict |
| Feature vs label? | Input variables বনাম target |
| Train/validation/test? | Learn / tune / final evaluate |
| Stratify কেন? | Class proportion preserve করতে |
| Time series split? | Temporal order preserve করতে |
| Overfitting? | Train ভালো, unseen খারাপ |
| Underfitting? | Train ও validation উভয়ই দুর্বল |
| Generalization? | Unseen data-তে performance |
| Leakage? | Prediction time-এ unavailable বা test-derived info ঢোকা |
| Baseline? | Simple comparison reference |
| Loss vs metric? | Optimize objective বনাম evaluate measure |
| Class imbalance? | Unequal class frequency |
| Threshold? | Score-কে class-এ map করার cutoff |
| Calibration? | Predicted probability-এর reliability |
| Batch vs online? | Scheduled retraining বনাম incremental update |

## Practice Questions

1. ML model আর rule-based system-এর পার্থক্য কী? একটি fraud detection example দিয়ে explain করো।
2. Supervised ও unsupervised learning-এর জন্য একটি করে use case দাও।
3. একই problem-কে classification ও regression হিসেবে কীভাবে frame করা যায়?
4. Test set দিয়ে বারবার tuning করলে কী সমস্যা?
5. Imputation split-এর আগে করলে leakage কেন হতে পারে?
6. Time-series data-তে random train/test split কেন ভুল হতে পারে?
7. একই customer-এর data একাধিক split-এ গেলে কী ধরনের leakage হতে পারে?
8. Baseline model কীভাবে choose করবে?
9. 99% accuracy-র fraud model কেন useless হতে পারে?
10. Precision/recall trade-off অনুযায়ী threshold কীভাবে select করবে?
11. Loss ও business evaluation metric আলাদা হতে পারে এমন example দাও।
12. Model production-এ degrade করলে কী কী monitor করবে?

## Practice করার পদ্ধতি

1. Model বেছে নেওয়ার আগে prediction target ও prediction time define করো।
2. Split strategy data generation process অনুযায়ী choose করো।
3. Preprocessing শুধু training data-তে fit করো।
4. Baseline ও candidate model একই evaluation strategy-তে compare করো।
5. Metric বেছে নেওয়ার সময় class imbalance ও business cost বিবেচনা করো।
