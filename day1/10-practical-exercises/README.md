# Day 1 Practical Exercises

এই chapter-এ Day 1-এর Python fundamentals, data structures, iterators, decorators, exceptions, asyncio, Pydantic, NumPy এবং Pandas একসঙ্গে practice করবে। লক্ষ্য শুধু code run করানো নয়; input contract, edge case, failure behavior, complexity এবং test-এর মাধ্যমে solution justify করা।

প্রতিটি exercise-এর জন্য:
1. Requirements পড়ে input/output contract লিখো।
2. Code লেখার আগে expected output predict করো।
3. Happy path ও edge case test করো।
4. কোন data structure বা API বেছে নিয়েছ এবং কেন, explain করো।

## Exercise 1: Word Frequency

একটি sentence থেকে প্রতিটি word কতবার এসেছে বের করো।

**Requirements**
- Case-insensitive হবে।
- Punctuation বাদ দেবে।
- Whitespace-এর একাধিক run handle করবে।
- সবচেয়ে বেশি ব্যবহৃত ৩টি word দেখাবে।
- একই frequency হলে tie-breaking policy define করবে।

**Example**

Input:

~~~text
Python, python! Data is useful. Data is powerful.
~~~

Expected counts:

~~~python
{"python": 2, "data": 2, "is": 1, "useful": 1, "powerful": 1}
~~~

Hint: `str.lower()`, `re`, `collections.Counter`।

**Edge cases:** empty string, punctuation-only input, mixed case, repeated whitespace।

## Exercise 2: Generator Pipeline

একটি generator pipeline তৈরি করো যা file থেকে line পড়ে:

1. Empty line বাদ দেবে।
2. Integer parse করবে।
3. শুধু positive number yield করবে।
4. Invalid integer-এর ক্ষেত্রে কী করবে, তা policy হিসেবে নির্ধারণ করবে: reject পুরো input, skip with logging, নাকি error record yield।

~~~mermaid
flowchart LR
    A["Text file"] --> B["Read line"]
    B --> C["Trim whitespace"]
    C --> D{"Empty?"}
    D -->|হ্যাঁ| B
    D -->|না| E["Parse integer"]
    E --> F{"Valid positive integer?"}
    F -->|হ্যাঁ| G["Yield number"]
    F -->|না| H["Apply invalid-row policy"]
    H --> B
~~~

**Example input**

~~~text
10

-3
abc
25
0
~~~

যদি invalid row skip করার policy নাও, expected output হবে `10, 25` এবং invalid line-এর count/log থাকবে।

**Edge cases:** missing file, invalid encoding, very large file, malformed line।

## Exercise 3: Timing Decorator

একটি decorator লেখো যা function কত সময় নিয়েছে তা measure করবে।

**Requirements**
- `time.perf_counter()` ব্যবহার করবে।
- `functools.wraps` যোগ করবে।
- Original function-এর args, kwargs ও return value preserve করবে।
- Original function exception raise করলে সেটি swallow করবে না।
- Timing output success ও failure উভয় ক্ষেত্রে print/log করবে।

**Test:** normal function, function with keyword arguments, function returning None এবং function raising ValueError।

## Exercise 4: Pydantic Request Model

একটি `PredictionRequest` model তৈরি করো (Pydantic v2):

- `features: list[float]`
- `model_name: str`
- `request_id: str`
- features খালি হতে পারবে না।
- model_name ও request_id whitespace-only হতে পারবে না।
- অতিরিক্ত field allow হবে কি না policy ঠিক করো।
- Strict parsing দরকার কি না সিদ্ধান্ত নাও।

**Test matrix**

| Input condition | Expected |
|---|---|
| Valid non-empty features | Accept |
| Empty features | Reject |
| Empty model_name | Reject |
| Missing request_id | Reject |
| Unexpected field | তোমার configured policy অনুযায়ী |
| Numeric string in features | Strict/coercion policy অনুযায়ী |

## Exercise 5: Async API Simulation

তিনটি async function লেখো, প্রতিটি আলাদা service-এর response simulate করবে।

**Requirements**
- Sequential এবং concurrent version implement করো।
- `asyncio.gather` দিয়ে results collect করো।
- Semaphore দিয়ে concurrency limit ২ করো।
- একটি service timeout করাও।
- একটি service exception raise করুক।
- Cancellation ও cleanup behavior observe করো।

~~~mermaid
flowchart TD
    A["Request batch"] --> B["Concurrency limiter"]
    B --> C["Service A"]
    B --> D["Service B"]
    B --> E["Service C"]
    C --> F["Collect results / errors"]
    D --> F
    E --> F
~~~

**Important:** Retry যোগ করলে idempotency ও duplicate side effect-এর ঝুঁকি বিবেচনা করো।

## Exercise 6: NumPy Statistics

একটি 2D array তৈরি করো যেখানে row হলো student এবং column হলো subject।

বের করো:
- প্রতিটি student-এর average।
- প্রতিটি subject-এর average।
- সর্বোচ্চ score।
- ৮০-এর বেশি score-এর count।
- Missing score থাকলে policy অনুযায়ী mean calculate করো।

**Requirements**
- `shape`, `ndim`, `dtype` inspect করো।
- প্রতিটি aggregation-এর output shape আগে predict করো।
- `axis=0` এবং `axis=1` explain করো।
- Broadcasting দিয়ে subject-wise bonus যোগ করো।

## Exercise 7: Pandas Transaction Analysis

~~~python
import pandas as pd

df = pd.DataFrame([
    {"id": 1, "customer": "A", "amount": 100, "status": "success", "month": "2026-01"},
    {"id": 2, "customer": "B", "amount": 250, "status": "failed",  "month": "2026-01"},
    {"id": 3, "customer": "A", "amount": 300, "status": "success", "month": "2026-02"},
    {"id": 4, "customer": "C", "amount": 150, "status": "success", "month": "2026-02"},
    {"id": 5, "customer": "B", "amount": 400, "status": "success", "month": "2026-02"},
])
~~~

Tasks:
1. Successful transaction filter করো।
2. Customer-wise total amount ও transaction count বের করো।
3. Month-wise total amount বের করো।
4. সবচেয়ে বেশি revenue করা customer বের করো।
5. Summary CSV export করো।
6. Duplicate transaction ID যোগ করে duplicate policy implement করো।
7. Missing customer এবং invalid amount যোগ করে rejected row report করো।
8. Customer dimension table-এর সঙ্গে `merge(validate=...)` ব্যবহার করে join করো।

**Acceptance:** Accepted row + rejected row count input row count-এর সঙ্গে reconcile করতে হবে।

## Exercise 8: Mini Data Pipeline

একটি ছোট deterministic pipeline তৈরি করো:

~~~mermaid
flowchart TD
    A["CSV file"] --> B["Load selected columns"]
    B --> C["Validate schema"]
    C --> D{"Valid?"}
    D -->|না| E["Raise meaningful error"]
    D -->|হ্যাঁ| F["Clean / transform"]
    F --> G["NumPy statistics"]
    G --> H["Summary JSON"]
    H --> I["Tests and report"]
~~~

**Requirements**
- Function ছোট ও single-purpose রাখবে।
- Type hints ব্যবহার করবে।
- Invalid input-এর জন্য meaningful custom exception দেবে।
- Logging যোগ করতে পারো, কিন্তু sensitive data log করবে না।
- একই input ও configuration দিলে deterministic result হবে।
- Input file path, output path এবং configuration function argument হিসেবে নেবে; hidden global state এড়িয়ে চলো।

Suggested function boundaries:
- `load_data()`
- `validate_data()`
- `clean_data()`
- `calculate_metrics()`
- `write_report()`

## Exercise 9: Data Structure and Complexity Challenge

একটি list of transaction IDs থেকে:
- Duplicate বাদ দিয়ে insertion order বজায় রাখো।
- বারবার membership check করো।
- List-based ও set-based approach compare করো।
- Build cost, lookup cost এবং memory trade-off explain করো।

**Example:** `[4, 2, 4, 1, 2, 5]` → `[4, 2, 1, 5]`

## Exercise 10: Reliability Challenge

একটি function বা small service boundary তৈরি করো যা:
- Input validate করে।
- Domain-specific exception raise করে।
- Resource context manager দিয়ে cleanup করে।
- Async dependency call করলে timeout ও bounded concurrency রাখে।
- Structured result বা error return করে।
- At least one failure scenario test করে।

এখানে লক্ষ্য সব concept জোর করে এক function-এ ঢোকানো নয়। ছোট module-এ responsibility ভাগ করো।

## Test Strategy

প্রতিটি exercise-এর জন্য অন্তত নিচের test category বিবেচনা করো:

| Test type | Example |
|---|---|
| Happy path | Valid input |
| Empty input | Empty string/list/file |
| Boundary | 0, 1, max allowed |
| Invalid type | String where integer expected |
| Missing data | Null / missing column |
| Duplicate | Repeated ID |
| Failure | File missing / API timeout |
| Determinism | Same input, same output |
| Cleanup | Exception হলেও resource release |

## Self-review checklist

- [ ] Code নিজে লিখেছি, শুধু copy করিনি।
- [ ] Input/output contract লিখেছি।
- [ ] Empty input handle করেছি।
- [ ] Invalid data handle করেছি।
- [ ] Function-এ type hints দিয়েছি।
- [ ] প্রয়োজনীয় জায়গায় specific exception handle করেছি।
- [ ] বড় input হলে memory impact ভেবেছি।
- [ ] Async task-এর timeout/cancellation বিবেচনা করেছি।
- [ ] Resource cleanup পরীক্ষা করেছি।
- [ ] অন্তত পাঁচটি automated test লিখেছি।
- [ ] Complexity ও trade-off explain করতে পারি।

## Completion Criteria

Exercise complete তখনই বলবে যখন:
1. Requirements পূরণ হয়।
2. Expected output ও edge case test pass করে।
3. Error handling স্পষ্ট।
4. Code readable ও function boundaries পরিষ্কার।
5. Design choice ও complexity নিজের ভাষায় explain করতে পারো।

