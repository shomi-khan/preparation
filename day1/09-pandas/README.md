# Pandas

বাস্তব data খুব কমই clean অবস্থায় পাওয়া যায়। Column-এর type ভুল হতে পারে, value missing থাকতে পারে, duplicate record থাকতে পারে, আর table-গুলোর মধ্যে relationship থাকতে পারে। **Pandas** tabular data load, inspect, clean, transform, join ও analyze করার জন্য Python-এর বহুল ব্যবহৃত library।

## ১. Pandas কী? Series এবং DataFrame কী?

Pandas-এর দুটি প্রধান data structure:

- **Series:** এক-dimensional labeled array। প্রতিটি value-এর সঙ্গে index label থাকে।
- **DataFrame:** দুই-dimensional labeled table; প্রতিটি column একটি Series-এর মতো।

~~~python
import pandas as pd

scores = pd.Series([80, 90, 75], index=["Mina", "Rafi", "Asha"])

students = pd.DataFrame({
    "name": ["Mina", "Rafi", "Asha"],
    "score": [80, 90, 75],
})

print(scores)
print(students)
~~~

~~~mermaid
flowchart TD
    A["DataFrame"] --> B["Index: row labels"]
    A --> C["Column: name → Series"]
    A --> D["Column: score → Series"]
~~~

## ২. CSV load এবং data inspect কীভাবে করব?

~~~python
df = pd.read_csv("transactions.csv")

print(df.head())
print(df.tail())
print(df.shape)
print(df.columns)
print(df.dtypes)
df.info()
print(df.describe(include="all"))
~~~

- `head()` / `tail()`: প্রথম/শেষ row দেখা।
- `shape`: row ও column count।
- `dtypes`: প্রতিটি column-এর data type।
- `info()`: non-null count, dtype ও memory summary।
- `describe()`: numeric বা selected column-এর summary statistics।

শুধু প্রথম কয়েকটি row দেখে data clean ধরে নিও না। Schema, missing value, unique value এবং domain-specific constraint-ও পরীক্ষা করো।

## ৩. Column select এবং row filter কীভাবে করব?

~~~python
amounts = df["amount"]                       # Series
subset = df[["customer_id", "amount"]]       # DataFrame

large = df[df["amount"] > 1000]
successful = df[df["status"] == "success"]

filtered = df[
    (df["amount"] > 1000) &
    (df["status"] == "success")
]
~~~

একাধিক condition-এ `&` এবং `|` ব্যবহার করতে হবে; প্রতিটি condition parentheses-এ রাখবে। Python-এর `and` / `or` সাধারণত Series-এর element-wise condition-এর জন্য নয়।

## ৪. loc এবং iloc-এর পার্থক্য কী?

- `loc`: index/column label দিয়ে select করে।
- `iloc`: integer position দিয়ে select করে।

~~~python
df = pd.DataFrame(
    {"name": ["Mina", "Rafi", "Asha"], "score": [80, 90, 75]},
    index=["u1", "u2", "u3"],
)

print(df.loc["u2", "score"])  # label
print(df.iloc[1, 1])          # row/column position
print(df.loc["u1":"u2"])      # label slice: endpoint included
print(df.iloc[0:2])           # position slice: endpoint excluded
~~~

~~~mermaid
flowchart LR
    A["Select data"] --> B["Know row/column label?"]
    B -->|হ্যাঁ| C["loc"]
    B -->|না, position জানা| D["iloc"]
~~~

Integer index থাকলেও `loc[1]` label 1 খোঁজে, দ্বিতীয় row নয়।

## ৫. Missing Values কীভাবে detect ও handle করব?

~~~python
print(df.isna().sum())

df["amount"] = df["amount"].fillna(0)
df = df.dropna(subset=["customer_id"])
~~~

Missing value fill বা drop করার আগে কেন missing হয়েছে এবং business meaning কী, তা বুঝতে হবে। Blindly zero fill করলে ভুল analysis হতে পারে। Numeric mean/median, category-এর mode, domain-specific default অথবা row removal-এর মধ্যে সিদ্ধান্ত data context-এর উপর নির্ভর করে।

**ML সতর্কতা:** Imputation rule train data-তে fit করে validation/test data-তে apply করো। পুরো dataset-এ আগে mean বের করলে data leakage হতে পারে।

## ৬. Duplicate Record কীভাবে handle করব?

~~~python
print(df.duplicated().sum())
df = df.drop_duplicates()

df = df.drop_duplicates(
    subset=["transaction_id"],
    keep="last",
)
~~~

কোন columns ধরে duplicate নির্ধারণ করবে তা স্পষ্ট করতে হবে। `keep="last"` কেবল তখনই যুক্তিযুক্ত যখন row ordering এবং latest record-এর অর্থ পরিষ্কার।

## ৭. Data Type Conversion কীভাবে করব?

~~~python
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")
df["created_at"] = pd.to_datetime(df["created_at"], errors="coerce")
~~~

`errors="coerce"` invalid value-কে missing value-তে রূপান্তর করে। Conversion-এর পরে নতুন NaN কত তৈরি হয়েছে তা inspect করো।

Date/time parsing-এ timezone, format এবং ambiguous date (যেমন 03/04/2026) সম্পর্কে স্পষ্ট policy থাকা দরকার।

## ৮. GroupBy এবং Aggregation কীভাবে কাজ করে?

~~~python
summary = (
    df.groupby("status")["amount"]
      .agg(["count", "sum", "mean", "max"])
      .reset_index()
)
~~~

এটি **split-apply-combine** pattern: data group-এ ভাগ হয়, প্রতিটি group-এ calculation হয়, তারপর result combine হয়।

~~~mermaid
flowchart LR
    A["DataFrame"] --> B["Split into groups"]
    B --> C["Apply aggregation"]
    C --> D["Combine results"]
    D --> E["Summary DataFrame"]
~~~

একাধিক column-এর named aggregation:

~~~python
summary = df.groupby("customer_id").agg(
    transaction_count=("transaction_id", "count"),
    total_amount=("amount", "sum"),
    average_amount=("amount", "mean"),
)
~~~

`count` non-missing value গোনে; `size` group-এর row count গোনে। Missing value থাকলে দুটির result আলাদা হতে পারে।

## ৯. Sorting কীভাবে করব?

~~~python
top_customers = summary.sort_values(
    "total_amount",
    ascending=False,
).head(10)
~~~

একাধিক column দিয়ে sort করা যায়:

~~~python
df.sort_values(
    ["status", "amount"],
    ascending=[True, False],
)
~~~

## ১০. Merge / Join কীভাবে কাজ করে?

দুটি DataFrame-এর common key দিয়ে row combine করতে `merge()` ব্যবহার হয়।

~~~python
transactions = pd.DataFrame({
    "customer_id": [1, 2],
    "amount": [100, 200],
})

customers = pd.DataFrame({
    "customer_id": [1, 2],
    "name": ["A", "B"],
})

result = transactions.merge(
    customers,
    on="customer_id",
    how="left",
    validate="many_to_one",
)
~~~

Common join types:
- `inner`: দুই table-এ matching key থাকা row।
- `left`: left table-এর সব row, match না থাকলে right columns missing।
- `right`: right table-এর সব row।
- `outer`: উভয় table-এর সব key।

~~~mermaid
flowchart TD
    A["Left DataFrame"] --> C["merge on key"]
    B["Right DataFrame"] --> C
    C --> D{"Join type"}
    D --> E["inner"]
    D --> F["left"]
    D --> G["right"]
    D --> H["outer"]
~~~

**Interview ও production point:** Duplicate join key result row অপ্রত্যাশিতভাবে বাড়িয়ে দিতে পারে। `validate="one_to_one"`, `"one_to_many"` বা `"many_to_one"` দিয়ে expected relationship enforce করো। Merge-এর আগে row count ও key uniqueness পরীক্ষা করো।

## ১১. concat এবং merge-এর পার্থক্য কী?

- `concat()`: DataFrame/Series-কে axis বরাবর জোড়া দেয়।
- `merge()`: relational key দিয়ে table join করে।

~~~python
combined_rows = pd.concat([january, february], ignore_index=True)
combined_columns = pd.concat([profile, metrics], axis=1)
~~~

দুটি table-এর matching ID দিয়ে relationship combine করতে `merge` ব্যবহার করো; একই schema-র monthly data stack করতে `concat` বেশি উপযোগী।

## ১২. Pivot Table কী?

Group এবং aggregation-কে row/column layout-এ reshape করতে `pivot_table()` ব্যবহার হয়।

~~~python
summary = pd.pivot_table(
    df,
    values="amount",
    index="customer_id",
    columns="status",
    aggfunc="sum",
    fill_value=0,
)
~~~

`pivot()` duplicate index/column combination পেলে error দেয়; `pivot_table()` aggregation function দিয়ে duplicate group combine করতে পারে।

## ১৩. Apply বনাম Vectorized Operation

সম্ভব হলে built-in vectorized operation ব্যবহার করো। Row-by-row `apply(axis=1)` অনেক ক্ষেত্রে ধীর হতে পারে।

~~~python
# Vectorized
df["amount_with_tax"] = df["amount"] * 1.15

# সাধারণত তুলনামূলকভাবে ধীর
# df["amount_with_tax"] = df.apply(
#     lambda row: row["amount"] * 1.15,
#     axis=1,
# )
~~~

Vectorized operation সাধারণত দ্রুত, কিন্তু complex logic-এর জন্য সবসময় vectorization সহজ নয়। Readability, memory এবং performance balance করো।

## ১৪. SettingWithCopyWarning কী? Chained assignment কেন এড়াব?

Filter করে পাওয়া object কখন view বা copy হবে তা operation ও pandas behavior-এর উপর নির্ভর করতে পারে। Chained assignment ambiguous এবং warning/ভুল update তৈরি করতে পারে।

এড়িয়ে চলো:

~~~python
# df[df["status"] == "success"]["amount"] = 0
~~~

স্পষ্টভাবে `loc` ব্যবহার করো:

~~~python
mask = df["status"] == "success"
df.loc[mask, "amount"] = 0
~~~

Independent subset দরকার হলে:

~~~python
subset = df.loc[mask].copy()
~~~

Pandas-এর Copy-on-Write behavior version অনুযায়ী বদলেছে; তবুও explicit selection ও assignment সবচেয়ে পরিষ্কার pattern।

## ১৫. Date/time data কীভাবে analyze করব?

~~~python
df["created_at"] = pd.to_datetime(df["created_at"], utc=True)

df["month"] = df["created_at"].dt.to_period("M")
monthly = df.groupby("month")["amount"].sum()
~~~

Timezone-aware timestamp ব্যবহার করলে UTC conversion ও local business timezone আলাদা করে বিবেচনা করো। Date range filter, resampling এবং time-based grouping-এর জন্য datetime dtype জরুরি।

## ১৬. CSV এবং JSON export কীভাবে করব?

~~~python
df.to_csv("cleaned.csv", index=False)
df.to_json("cleaned.json", orient="records", indent=2)
~~~

Export-এর আগে column order, null representation, date format, encoding এবং sensitive field বাদ দেওয়া হয়েছে কি না যাচাই করো।

## ১৭. বড় Dataset-এর ক্ষেত্রে কী করব?

- অপ্রয়োজনীয় column load করবে না: `usecols`।
- উপযুক্ত `dtype` ব্যবহার করবে।
- বড় CSV chunk করে পড়বে: `chunksize`।
- সম্ভব হলে vectorized operation ব্যবহার করবে।
- প্রয়োজন অনুযায়ী Parquet-এর মতো columnar format বিবেচনা করবে।
- Dataset RAM-এ না ধরলে database query বা distributed processing tool বিবেচনা করবে।

~~~python
total = 0

for chunk in pd.read_csv(
    "large.csv",
    usecols=["status", "amount"],
    chunksize=50_000,
):
    successful = chunk.loc[chunk["status"] == "success"]
    total += successful["amount"].sum()

print(total)
~~~

Chunk processing-এ aggregation এমন হতে হবে যা chunk-wise result combine করা যায়। Median/quantile-এর মতো operation-এর জন্য naive partial aggregation যথেষ্ট নাও হতে পারে।

## ১৮. AI/ML workflow-এ Pandas-এর ভূমিকা কী?

- Raw dataset inspect করা।
- Missing ও invalid data handle করা।
- Feature তৈরি করা।
- Label distribution দেখা।
- Group-wise analysis করা।
- Model training-এর আগে data প্রস্তুত করা।

Train/test split-এর আগে পুরো dataset-এ target-informed preprocessing করলে leakage হতে পারে। Imputation, scaling ও feature selection-এর মতো learned transformation train data-তে fit করে validation/test data-তে apply করো।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Series বনাম DataFrame? | 1D labeled array বনাম 2D labeled table |
| loc বনাম iloc? | Label-based বনাম position-based |
| Missing value detect? | isna() |
| count বনাম size? | Non-null count বনাম row count |
| groupby pattern? | Split-apply-combine |
| merge বনাম concat? | Key-based join বনাম axis-wise combine |
| validate= কেন? | Join cardinality enforce |
| pivot বনাম pivot_table? | Reshape unique cells বনাম aggregate duplicates |
| Chained assignment? | Ambiguous; loc ব্যবহার |
| apply(axis=1) কেন ধীর? | Python-level row iteration |
| chunksize? | Chunk-by-chunk file processing |
| Data leakage? | Validation/test information training transformation-এ ঢুকে পড়া |

## Practice Exercises

একটি transaction dataset নিয়ে:

1. Dataset load ও inspect করো।
2. Duplicate transaction remove করো।
3. Missing `customer_id` বাদ দেওয়ার আগে missing rate report করো।
4. `amount` numeric-এ convert করো এবং invalid value identify করো।
5. Successful transaction filter করো।
6. Customer-wise total amount ও transaction count বের করো।
7. Top 10 customer বের করো।
8. Monthly transaction summary তৈরি করো।
9. Customer table-এর সঙ্গে merge করে `validate` ব্যবহার করো।
10. Status বনাম month pivot table তৈরি করো।
11. Chunked processing দিয়ে successful transaction total বের করো।
12. Cleaning-এর আগে/পরে row count reconcile করো এবং rejected row-এর কারণ report করো।

## Practice করার পদ্ধতি

1. Data transform করার আগে row count, column type ও missing count লিখে রাখো।
2. প্রতিটি cleaning decision-এর business reason document করো।
3. Join-এর আগে key uniqueness ও expected cardinality verify করো।
4. ML preprocessing-এ train/test leakage এড়াও।
5. Output validation করো: row count, totals, nulls এবং expected uniqueness।
