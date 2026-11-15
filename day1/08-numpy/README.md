# NumPy

Machine learning ও scientific computing-এ data প্রায়ই vector, matrix বা higher-dimensional tensor হিসেবে থাকে। Python list দিয়ে এগুলো represent করা যায়, কিন্তু numerical operation-এ list নিজে mathematical array-এর মতো behave করে না। **NumPy** এই জায়গায় typed N-dimensional array এবং optimized numerical operation দেয়।

## ১. NumPy কী?

NumPy Python-এর numerical computing library। এর প্রধান data structure হলো homogeneous, N-dimensional `ndarray`। Matrix operation, numerical transformation এবং ML data preparation-এ এটি বহুল ব্যবহৃত।

~~~bash
pip install numpy
~~~

~~~python
import numpy as np

a = np.array([1, 2, 3])
matrix = np.array([[1, 2], [3, 4]])

print(a)
print(matrix)
~~~

## ২. ndarray-এর shape, dimension, size এবং dtype কী?

~~~python
x = np.array([[1, 2, 3], [4, 5, 6]])

print(x.shape)  # (2, 3)
print(x.ndim)   # 2
print(x.size)   # 6
print(x.dtype)  # integer dtype
~~~

- `shape`: প্রতিটি axis-এর size।
- `ndim`: axis বা dimension সংখ্যা।
- `size`: মোট element।
- `dtype`: element-এর data type।

~~~mermaid
flowchart TD
    A["Array shape: (2, 3)"] --> B["2 rows"]
    A --> C["3 columns"]
    A --> D["ndim = 2"]
    A --> E["size = 2 × 3 = 6"]
~~~

## ৩. NumPy array-এর dtype কেন গুরুত্বপূর্ণ?

একটি সাধারণ NumPy array-তে element-গুলো একই dtype-এ store হয়। এটি memory efficiency ও fast numerical operation-এ সাহায্য করে।

~~~python
integers = np.array([1, 2, 3], dtype=np.int32)
floats = integers.astype(np.float64)

print(integers.dtype)
print(floats.dtype)
~~~

Mixed type input দিলে NumPy common dtype বেছে নিতে পারে। Conversion-এ precision loss বা overflow হতে পারে, তাই data type সচেতনভাবে নির্বাচন করো।

## ৪. Array তৈরি করার common উপায় কী?

~~~python
zeros = np.zeros((2, 3))
ones = np.ones((2, 2))
sequence = np.arange(0, 10, 2)
points = np.linspace(0, 1, 5)
identity = np.eye(3)

print(zeros)
print(sequence)  # [0 2 4 6 8]
print(points)    # 0 থেকে 1 inclusive, 5টি value
~~~

- `zeros`, `ones`: নির্দিষ্ট shape-এর initialized array।
- `arange`: step অনুযায়ী sequence।
- `linspace`: নির্দিষ্ট সংখ্যক evenly spaced point।
- `eye`: identity matrix।

Floating-point step-এর ক্ষেত্রে `arange` endpoint behavior নিয়ে সতর্ক থাকো; নির্দিষ্ট সংখ্যক point চাইলে `linspace` বেশি উপযোগী।

## ৫. Indexing এবং Slicing কীভাবে কাজ করে?

~~~python
x = np.array([[10, 20, 30], [40, 50, 60]])

print(x[0, 1])   # 20
print(x[:, 1])   # দ্বিতীয় column
print(x[1, :2])  # দ্বিতীয় row-এর প্রথম দুইটি item
~~~

Basic slicing প্রায়ই view দেয়, copy নয়। তাই slice পরিবর্তন করলে original array-ও পরিবর্তিত হতে পারে।

~~~mermaid
flowchart LR
    A["Original array"] --> B["Basic slice / view"]
    B --> C["Shares underlying data"]
    C --> D["Mutation may affect original"]
~~~

~~~python
x = np.array([10, 20, 30, 40])
view = x[1:3]
view[0] = 999
print(x)  # [10, 999, 30, 40]
~~~

Independent copy দরকার হলে `copy()` ব্যবহার করো।

~~~python
independent = x[1:3].copy()
~~~

## ৬. Basic indexing ও advanced indexing-এর পার্থক্য কী?

Integer array বা boolean mask দিয়ে indexing করলে সাধারণত নতুন copy পাওয়া যায়, basic slicing-এর মতো view নয়।

~~~python
x = np.array([10, 20, 30, 40])
selected = x[[0, 2]]
selected[0] = 999

print(x)         # original unchanged
print(selected)  # [999, 30]
~~~

Boolean indexing দিয়ে condition অনুযায়ী item select করা যায়:

~~~python
values = np.array([3, 8, 1, 10, 5])
print(values[values > 5])  # [8, 10]
~~~

## ৭. Vectorization কী?

Python loop দিয়ে প্রতিটি element process করার বদলে NumPy array operation ব্যবহার করে পুরো array-এর উপর operation করা যায়। এতে Python-level loop কমে এবং optimized native implementation কাজে লাগে।

~~~python
prices = np.array([100, 200, 300])
discounted = prices * 0.9
print(discounted)
~~~

~~~mermaid
flowchart TD
    A["Python list values"] --> B["Python loop per element"]
    A --> C["Convert to ndarray"]
    C --> D["Vectorized array operation"]
    D --> E["Optimized numerical implementation"]
~~~

Vectorization সাধারণত দ্রুত হতে পারে, কিন্তু temporary intermediate array memory খরচ বাড়াতে পারে। সব workload-এ automatic speedup হবে না।

## ৮. Broadcasting কী?

Broadcasting compatible shape-এর array-এর মধ্যে operation করার নিয়ম। ছোট shape-এর array-কে conceptually বড় shape-এর সঙ্গে align করা হয়; সাধারণত actual repeated copy তৈরি করা হয় না।

~~~python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

bias = np.array([10, 20, 30])
print(matrix + bias)
~~~

Broadcasting-এর নিয়ম: দুই shape-এর dimension ডান দিক থেকে compare করা হয়। প্রতিটি aligned dimension হয় সমান হতে হবে, নয়তো কোনো একটির size 1 হতে হবে। Missing leading dimension-কে 1 ধরা হয়।

~~~mermaid
flowchart TD
    A["Matrix shape (2, 3)"] --> C["Align from right"]
    B["Vector shape (3,)"] --> C
    C --> D["Compare: (2,3) and (1,3)"]
    D --> E{"Each pair equal or one is 1?"}
    E -->|হ্যাঁ| F["Broadcast result shape (2,3)"]
    E -->|না| G["Shape mismatch error"]
~~~

## ৯. axis=0 এবং axis=1 কী বোঝায়?

~~~python
scores = np.array([[80, 90], [70, 85]])

print(scores.mean())         # all elements
print(scores.mean(axis=0))   # column-wise: [75., 87.5]
print(scores.mean(axis=1))   # row-wise: [85., 77.5]
~~~

Reduction-এ যে axis collapse হয়, সেটি result থেকে বাদ যায়। 2D array-তে:
- `axis=0`: row direction collapse হয়, column-wise result থাকে।
- `axis=1`: column direction collapse হয়, row-wise result থাকে।

~~~mermaid
flowchart TD
    A["2D array: (rows, columns)"] --> B["sum(axis=0)"]
    A --> C["sum(axis=1)"]
    B --> D["Collapse rows → one value per column"]
    C --> E["Collapse columns → one value per row"]
~~~

## ১০. Reshape কী? এটি কি copy তৈরি করে?

~~~python
values = np.arange(12)
matrix = values.reshape(3, 4)
print(matrix)
~~~

Reshape করতে element count একই থাকতে হবে। সম্ভব হলে reshape view দেয়; memory layout-এর কারণে copy লাগতে পারে।

~~~python
print(np.shares_memory(values, matrix))
~~~

`reshape(-1)` একটি dimension flatten করে। `ravel()` সম্ভব হলে view দেয়, আর `flatten()` সবসময় copy return করে।

## ১১. Transpose কী?

Transpose array-এর axis order বদলায়। 2D matrix-এর ক্ষেত্রে row ও column অদলবদল হয়।

~~~python
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print(matrix.T)
print(matrix.T.shape)  # (3, 2)
~~~

Transpose অনেক ক্ষেত্রে data copy না করে strides/shape metadata বদলে view দিতে পারে। Non-contiguous layout পরবর্তী operation-এর performance-এ প্রভাব ফেলতে পারে।

## ১২. Element-wise multiplication এবং Matrix multiplication-এর পার্থক্য কী?

~~~python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

print(a * b)  # element-wise
print(a @ b)  # matrix multiplication
# np.matmul(a, b) equivalent matrix multiplication
~~~

Matrix multiplication-এর জন্য left matrix-এর শেষ dimension এবং right matrix-এর শেষ থেকে দ্বিতীয় dimension compatible হতে হবে।

~~~mermaid
flowchart LR
    A["A: (m, n)"] --> C["A @ B"]
    B["B: (n, p)"] --> C
    C --> D["Result: (m, p)"]
~~~

## ১৩. Linear algebra-এ solve বনাম inverse কী?

যদি `A @ x = b` solve করতে হয়, সাধারণত explicit inverse `inv(A) @ b` করার বদলে `np.linalg.solve(A, b)` ব্যবহার করা ভালো।

~~~python
A = np.array([[3.0, 1.0], [1.0, 2.0]])
b = np.array([9.0, 8.0])

x = np.linalg.solve(A, b)
print(x)
~~~

`solve` numerical algorithm দিয়ে system solve করে; explicit inverse সাধারণত অপ্রয়োজনীয় এবং numerical stability/performance খারাপ করতে পারে।

## ১৪. NaN এবং missing numerical value কীভাবে handle করব?

Floating-point array-এ missing/invalid numerical value represent করতে `np.nan` ব্যবহার হয়।

~~~python
values = np.array([10.0, np.nan, 30.0])

print(np.isnan(values))
print(np.nanmean(values))  # 20.0
~~~

`np.nan == np.nan` False, তাই NaN check করতে `np.isnan()` ব্যবহার করো। `nanmean`, `nansum`-এর মতো function missing value ignore করতে পারে; data quality policy বুঝে ব্যবহার করো।

## ১৫. Boolean masking ও np.where কী?

~~~python
scores = np.array([45, 70, 88, 32])
passed = scores >= 50

print(scores[passed])  # [70 88]
print(np.where(passed, "pass", "fail"))
~~~

Mask দিয়ে filter করা যায়; `np.where(condition, x, y)` condition অনুযায়ী element select করে।

## ১৬. Random Data এবং Reproducibility কীভাবে করব?

~~~python
rng = np.random.default_rng(seed=42)
samples = rng.normal(loc=0, scale=1, size=5)
print(samples)
~~~

Modern NumPy-তে `default_rng()` recommended API। Seed নির্দিষ্ট করলে reproducible workflow সহজ হয়। Exact sequence library version বদলালে বদলাতে পারে; reproducibility-এর জন্য environment ও version-ও record করো।

## ১৭. Python List বনাম NumPy Array

| বিষয় | Python List | NumPy Array |
|---|---|---|
| Data type | বিভিন্ন type থাকতে পারে | সাধারণত homogeneous dtype |
| Numerical operation | loop/comprehension | vectorized operation |
| Multidimensional data | nested list | native N-D array |
| Memory layout | object reference | typed storage |
| Use case | general collection | numerical computation |

NumPy সব ক্ষেত্রে list-এর চেয়ে দ্রুত নয়; operation, data size, dtype এবং conversion cost গুরুত্বপূর্ণ।

## ১৮. AI/ML-এ NumPy কেন গুরুত্বপূর্ণ?

- Dataset numerical array-এ represent করা।
- Feature scaling ও transformation।
- Matrix ও vector operation।
- Model-এর input/output process করা।
- Statistical calculation।
- Data shape ও batch dimension reason করা।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| ndarray কী? | Homogeneous N-dimensional array |
| shape, ndim, size? | Axis sizes, axis count, total elements |
| dtype কেন গুরুত্বপূর্ণ? | Memory, precision ও operation behavior |
| Vectorization? | Array operation দিয়ে Python loop কমানো |
| Broadcasting? | Compatible shape-এর array operation |
| axis=0? | Row axis collapse, column-wise result |
| axis=1? | Column axis collapse, row-wise result |
| Basic slice? | সাধারণত view |
| Advanced indexing? | সাধারণত copy |
| reshape সবসময় copy? | না, view হতে পারে |
| ravel বনাম flatten? | ravel view দিতে পারে; flatten copy |
| * বনাম @? | Element-wise বনাম matrix multiplication |
| NaN check? | np.isnan() |
| solve বনাম inv? | Linear system solve করতে solve() prefer |
| default_rng? | Modern random Generator API |

## Practice Exercises

1. ১ থেকে ২০ পর্যন্ত integer-এর array তৈরি করো এবং dtype inspect করো।
2. ৩x৩ matrix তৈরি করে row-wise ও column-wise sum বের করো।
3. Broadcasting দিয়ে প্রতিটি row-তে bias যোগ করো। Shape আগে predict করো।
4. Basic slice mutate করে original array change হয় কি না পরীক্ষা করো।
5. Advanced indexing copy দেয় কি না পরীক্ষা করো।
6. `reshape`, `ravel` ও `flatten`-এর memory sharing compare করো।
7. দুইটি compatible matrix multiply করো এবং result shape predict করো।
8. `np.linalg.solve` দিয়ে linear system solve করো।
9. NaN থাকা array-এর mean ও nanmean compare করো।
10. Python list loop এবং NumPy vectorization-এর execution time তুলনা করো। Conversion cost-ও include করো।
11. `default_rng` দিয়ে reproducible sample তৈরি করো।
12. (100, 4) feature matrix ও (4, 2) weight matrix নিয়ে `X @ W` করো এবং shape explain করো।

## Practice করার পদ্ধতি

1. Operation চালানোর আগে input ও output shape predict করো।
2. View নাকি copy তৈরি হচ্ছে, mutation দিয়ে পরীক্ষা করো।
3. dtype, precision, NaN এবং memory layout বিবেচনা করো।
4. Performance compare করার সময় data size ও conversion overhead account করো।
5. ML-এ প্রতিটি transformation-এর আগে ও পরে feature dimension লিখে রাখো।
