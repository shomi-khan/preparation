# Day 2 Practical Exercises

এই chapter-এর লক্ষ্য শুধু model train করা নয়। একটি ML problem define করা, data split ও leakage control করা, baseline-এর সঙ্গে model compare করা এবং result explain করতে পারা।

প্রতিটি exercise-এর জন্য:
1. Problem ও prediction time লিখে নাও।
2. Input feature এবং target define করো।
3. Split strategy justify করো।
4. Expected result predict করে code run করো।
5. Metric, error এবং limitation explain করো।

## Exercise 1: Problem Classification

নিচের প্রতিটি problem supervised classification, regression, clustering, self-supervised নাকি reinforcement learning-এর কাছাকাছি, তা চিহ্নিত করো এবং কারণ লেখো:

1. Email spam detection।
2. আগামী মাসের electricity demand predict।
3. Customer segmentation।
4. Loan default risk prediction।
5. Used car price estimation।
6. Large text corpus থেকে next-token prediction।
7. Robot reward maximize করে navigation শেখে।

## Exercise 2: Data Exploration

একটি tabular dataset বেছে নাও। বের করো:
- Row/column count।
- Data types।
- Missing value count ও percentage।
- Duplicate row count।
- Target class distribution।
- Numeric feature summary।
- Potential leakage column।
- ID/group/time dependency।

**Deliverable:** Data quality report এবং কোন column feature হিসেবে বাদ দেবে তার কারণ।

## Exercise 3: Train/Test Split Strategy

একটি classification dataset-এ stratified split করো। Split-এর আগে ও পরে class distribution compare করো।

তারপর নিচের scenario-গুলোর জন্য split strategy select করো:
- IID customer transaction।
- একই customer-এর বহু transaction।
- Time-ordered sensor readings।
- Patient-এর multiple visit।

**Explain:** Random split, stratified split, group split এবং time-based split-এর trade-off।

## Exercise 4: Baseline Model

DummyClassifier এবং LogisticRegression train করো। Accuracy নয়, F1, precision, recall ও confusion matrix তুলনা করো।

**Questions:**
- Baseline-এর চেয়ে উন্নতি হয়েছে কি?
- Minority class-এর performance কেমন?
- Model কোন ধরনের ভুল বেশি করছে?

## Exercise 5: Model Comparison

একই split ও preprocessing strategy ব্যবহার করে:
- Logistic Regression
- Decision Tree
- Random Forest
- Gradient Boosting classifier (যদি library available থাকে)

তুলনা করো:
- Cross-validation score।
- Training time।
- Inference time।
- Model complexity।
- Interpretability।
- Failure mode।

একই validation strategy ছাড়া score compare কোরো না।

## Exercise 6: Leakage Demonstration

একটি dataset-এ দুই পদ্ধতি compare করো:

1. পুরো data-তে scaler fit করে তারপর split।
2. Split-এর পরে Pipeline ব্যবহার করে scaler train data-তে fit।

দেখাও test data-র distribution training preprocessing-এ ঢুকলে কী ধরনের leakage হতে পারে। তারপর Pipeline + cross-validation ব্যবহার করে সঠিক workflow দেখাও।

## Exercise 7: Imbalanced Dataset

একটি imbalanced classification dataset নাও।

- Accuracy, precision, recall, F1 ও PR-AUC তুলনা করো।
- Confusion matrix analyze করো।
- Class weight বা resampling experiment করো।
- False negative ও false positive-এর business cost লিখো।

**Challenge:** Validation data-তে threshold tune করো। Test set untouched রাখো।

## Exercise 8: Cross-validation and Hyperparameter Tuning

1. StratifiedKFold দিয়ে cross-validation চালাও।
2. GridSearchCV দিয়ে Logistic Regression-এর C tune করো।
3. RandomizedSearchCV-এর সঙ্গে compare করো।
4. Mean score-এর পাশাপাশি fold-to-fold variation report করো।
5. Data group/time-dependent হলে standard KFold কেন ভুল হতে পারে explain করো।

## Exercise 9: Calibration and Threshold

একটি classifier-এর predicted probability নাও:
- Calibration curve plot করো।
- Brier score বের করো।
- Default threshold এবং chosen threshold-এর precision/recall compare করো।
- Threshold নির্বাচন করার business rationale লিখো।

Threshold ও calibration final test set দিয়ে tune কোরো না।

## Exercise 10: Model Artifact

Best model joblib দিয়ে save করো, reload করো এবং held-out sample-এ prediction compare করো।

Record করো:
- Model version।
- Dataset version।
- Feature schema।
- Python ও package version।
- Training configuration।
- Evaluation metric।
- Threshold।
- Artifact checksum বা registry ID।

শুধু trusted artifact load করবে।

## Exercise 11: Production Monitoring Scenario

ধরো deployed fraud model-এর:
- Input missing rate বেড়েছে।
- Prediction score distribution বদলেছে।
- Ground-truth fraud label দুই সপ্তাহ পরে আসে।
- API latency বেড়েছে।

প্রতিটি signal-এর জন্য metric, alert এবং investigation plan define করো। Data drift ও concept drift আলাদা করে explain করো।

## Exercise 12: End-to-End Mini Project

একটি dataset নিয়ে reproducible ML project তৈরি করো।

~~~mermaid
flowchart TD
    A["Problem definition"] --> B["Data inspection"]
    B --> C["Leakage-aware split"]
    C --> D["Baseline"]
    D --> E["Preprocessing Pipeline"]
    E --> F["Cross-validation and tuning"]
    F --> G["Validation threshold / calibration"]
    G --> H["Final test evaluation"]
    H --> I["Save artifact and metadata"]
    I --> J["Monitoring plan"]
~~~

**Deliverables**
- Notebook বা Python project।
- Dataset description ও target definition।
- Split strategy explanation।
- Baseline ও model comparison table।
- Confusion matrix ও metric report।
- Leakage prevention explanation।
- Saved model artifact এবং metadata।
- ৩-৫ মিনিটের project walkthrough।

## Self-review

- [ ] Target ও prediction time সঠিকভাবে define করেছি।
- [ ] Leakage column inspect করেছি।
- [ ] Appropriate split strategy বেছে নিয়েছি।
- [ ] Preprocessing Pipeline-এ রেখেছি।
- [ ] Baseline-এর সঙ্গে compare করেছি।
- [ ] Imbalanced data-তে উপযুক্ত metric বেছে নিয়েছি।
- [ ] Test set tuning-এ ব্যবহার করিনি।
- [ ] Threshold validation data-তে select করেছি।
- [ ] Artifact ও dependency version record করেছি।
- [ ] Limitation ও monitoring plan লিখেছি।

## Completion Criteria

Project complete তখনই বলবে যখন:
1. Problem framing ও split strategy explain করতে পারো।
2. Leakage-aware pipeline কাজ করে।
3. Baseline-এর সঙ্গে fair comparison আছে।
4. Metric business objective-এর সঙ্গে aligned।
5. Final test evaluation untouched data-তে হয়েছে।
6. Model reload করে inference করা যায়।
7. Production failure ও monitoring plan explain করতে পারো।
