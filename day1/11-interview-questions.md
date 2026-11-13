# Day 1 Interview Questions and Answers

## Python Fundamentals

### ১. Python কি interpreted নাকি compiled language?

Python-কে সাধারণত interpreted language বলা হয়। তবে CPython implementation source code-কে bytecode-এ compile করে, এরপর virtual machine bytecode execute করে। তাই "শুধু interpreted" বলা পুরো implementation process ব্যাখ্যা করে না।

### ২. Mutable এবং Immutable object-এর পার্থক্য কী?

Mutable object-এর state পরিবর্তন করা যায়, যেমন list ও dictionary। Immutable object-এর value পরিবর্তন করা যায় না, যেমন int ও string। Mutable object function-এ pass করলে function সেটিকে mutate করতে পারে।

### ৩. `is` এবং `==` কখন ব্যবহার করবে?

Value equality-এর জন্য `==`; object identity-এর জন্য `is`। `None` check করতে `is None` ব্যবহার করা উচিত।

### ৪. Shallow copy এবং Deep copy-এর পার্থক্য কী?

Shallow copy নতুন outer container তৈরি করে, কিন্তু nested object-এর reference share করতে পারে। Deep copy nested object-ও recursively copy করে। Deep copy বেশি memory/time নিতে পারে।

### ৫. Python-এ mutable default argument কেন বিপজ্জনক?

Default argument function definition-এর সময় একবার evaluate হয়। তাই default list বা dictionary পরবর্তী call-গুলোর মধ্যে share হতে পারে। Default হিসেবে None নিয়ে function-এর ভেতরে নতুন object তৈরি করা নিরাপদ pattern।

### ৬. GIL কী?

CPython-এর Global Interpreter Lock এমন একটি mechanism যা সাধারণত একটি interpreter process-এ এক সময়ে একটি thread-কে Python bytecode execute করতে দেয়। ফলে CPU-bound pure Python thread সাধারণত একাধিক core-এ parallel speedup পায় না। I/O-bound thread ও GIL release করা native operation-এ concurrency উপকারী হতে পারে। Python-এর implementation/build অনুযায়ী বিস্তারিত আচরণ ভিন্ন হতে পারে।

### ৭. `*args` এবং `**kwargs` কী?

`*args` অতিরিক্ত positional argument tuple হিসেবে এবং `**kwargs` অতিরিক্ত keyword argument dictionary হিসেবে সংগ্রহ করে।

### ৮. List comprehension কেন ব্যবহার করবে?

Iterable transform/filter করে concise list তৈরি করতে। তবে nested বা জটিল logic হলে normal loop বেশি readable হতে পারে।

## Data Structures

### ৯. List এবং Tuple-এর পার্থক্য কী?

List mutable; tuple immutable। List পরিবর্তনশীল sequence-এর জন্য, tuple fixed record বা অপরিবর্তনীয় sequence-এর জন্য উপযোগী।

### ১০. List-এর বদলে Set কখন ব্যবহার করবে?

Unique value দরকার হলে বা বারবার membership check করতে হলে set উপযোগী। Set unordered collection; positional indexing সমর্থন করে না।

### ১১. Dictionary lookup সাধারণত O(1) কেন?

Dictionary hash table ব্যবহার করে। Key-এর hash থেকে সম্ভাব্য storage location নির্ধারণ করা হয়। Average lookup O(1), তবে collision ও pathological case-এ worst case O(n) হতে পারে।

### ১২. Stack এবং Queue-এর পার্থক্য কী?

Stack LIFO; Queue FIFO। Python-এ stack-এর জন্য list এবং efficient queue-এর জন্য `collections.deque` ব্যবহার করা যায়।

## Iterators and Generators

### ১৩. Iterable এবং Iterator-এর পার্থক্য কী?

Iterable থেকে `iter()` দিয়ে iterator পাওয়া যায়। Iterator `next()` দিয়ে একে একে item দেয় এবং exhausted হলে StopIteration raise করে।

### ১৪. Generator কী?

`yield`-যুক্ত function call করলে generator object পাওয়া যায়। এটি execution state ধরে রেখে lazyভাবে item produce করে।

### ১৫. Generator কেন memory efficient হতে পারে?

সব result একসঙ্গে memory-তে না রেখে প্রয়োজন অনুযায়ী produce করে। বড় file বা stream process করতে উপকারী।

### ১৬. Generator কি পুনরায় ব্যবহার করা যায়?

একটি generator সাধারণত একবার consume করা যায়। Exhausted হলে নতুন generator তৈরি করতে হয়।

## Decorators and Context Managers

### ১৭. Decorator কী?

Decorator একটি function বা class-কে wrap করে অতিরিক্ত behavior যোগ করে, মূল implementation না বদলিয়ে। Logging, timing, caching ও authorization-এ ব্যবহার হয়।

### ১৮. `functools.wraps` কেন ব্যবহার করা হয়?

Wrapper-এর মধ্যে original function-এর metadata যেমন name ও docstring preserve করতে।

### ১৯. Context Manager কী?

Resource setup ও cleanup manage করে। `with` block থেকে বের হলে cleanup হয়, exception ঘটলেও। File handle, lock ও database session-এ এটি কাজে লাগে।

### ২০. `__enter__` ও `__exit__` কী করে?

`__enter__` resource acquire/setup করে এবং `as` variable-এ যে object bind হবে তা return করে। `__exit__` block শেষে cleanup করে এবং exception suppress করা হবে কি না নির্ধারণ করতে পারে।

## Exceptions and Type Hints

### ২১. `finally` কেন ব্যবহার করা হয়?

Cleanup বা এমন কাজের জন্য যা সাধারণত success/failure উভয় flow-তেই চালানো দরকার। যেমন resource release। Process forcibly terminate হলে সবসময় চলবে, এমন guarantee নেই।

### ২২. কেন broad exception catch করা উচিত নয়?

এতে programming bug আড়াল হতে পারে এবং failure-এর কারণ বোঝা কঠিন হয়। Expected exception নির্দিষ্টভাবে handle করা ভালো।

### ২৩. Type Hints কি runtime-এ enforce হয়?

না, Python নিজে সাধারণভাবে type hints enforce করে না। এগুলো IDE ও static type checker-কে সাহায্য করে। Runtime validation-এর জন্য Pydantic-এর মতো tool প্রয়োজন হতে পারে।

## Asyncio

### ২৪. Concurrency এবং Parallelism-এর পার্থক্য কী?

Concurrency একাধিক কাজের progress interleave করে; parallelism একাধিক কাজ একই সময়ে execute করে। Asyncio concurrency দেয়, কিন্তু নিজে থেকে CPU parallelism দেয় না।

### ২৫. Asyncio কখন ব্যবহার করবে?

Network request, async database driver বা অন্য I/O-bound operation-এ অপেক্ষার সময় অন্য task চালাতে। CPU-heavy কাজের জন্য সাধারণত process pool বা উপযুক্ত native library বিবেচনা করা হয়।

### ২৬. Event Loop কী?

Event loop coroutine ও I/O event schedule করে। Coroutine await-এ suspend হলে অন্য ready task চালাতে পারে।

### ২৭. Async function-এর ভেতরে blocking call করলে কী সমস্যা?

Blocking call event loop thread আটকে দিতে পারে। ফলে অন্য coroutine-ও progress করতে পারে না। Async-compatible library বা thread executor ব্যবহার করা যায়।

## Pydantic

### ২৮. Pydantic কেন ব্যবহার করবে?

Python type hints ও validation rule ব্যবহার করে external input parse/validate করতে। API request, configuration ও structured LLM output validate করতে উপকারী।

### ২৯. Pydantic কি authorization নিশ্চিত করে?

না। এটি data validation করে; user কোন resource access করতে পারবে তা authorization layer-এ enforce করতে হয়।

## NumPy

### ৩০. NumPy Array এবং Python List-এর পার্থক্য কী?

NumPy array সাধারণত homogeneous typed data compactভাবে রাখে এবং vectorized numerical operation দেয়। Python list general-purpose collection এবং বিভিন্ন type রাখতে পারে।

### ৩১. Vectorization কী?

Element-by-element Python loop না লিখে array operation দিয়ে অনেক value একসঙ্গে process করা। এতে optimized native code ব্যবহার হওয়ায় প্রায়ই performance বাড়ে।

### ৩২. Broadcasting কী?

Compatible shape-এর array-কে element-wise operation করতে দেওয়ার নিয়ম, যেখানে ছোট array conceptually বড় shape-এর সঙ্গে align হয়; অপ্রয়োজনীয় full copy সবসময় তৈরি হয় না।

### ৩৩. `axis=0` এবং `axis=1` কী বোঝায়?

2D array-তে `axis=0` row direction collapse করে column-wise result দেয়; `axis=1` column direction collapse করে row-wise result দেয়।

### ৩৪. `a * b` এবং `a @ b`-এর পার্থক্য কী?

`a * b` element-wise multiplication; `a @ b` matrix multiplication।

## Pandas

### ৩৫. Series এবং DataFrame-এর পার্থক্য কী?

Series এক-dimensional labeled data; DataFrame দুই-dimensional tabular data।

### ৩৬. Missing value কীভাবে handle করবে?

প্রথমে missing-এর কারণ ও business meaning বুঝব। তারপর drop, fill, imputation বা explicit missing category ব্যবহার করব। সব ক্ষেত্রে zero fill করা সঠিক নয়।

### ৩৭. GroupBy কীভাবে কাজ করে?

Split-apply-combine pattern: data group করা হয়, প্রতিটি group-এ aggregation/transformation চালানো হয়, তারপর result combine হয়।

### ৩৮. Pandas-এ merge কেন ব্যবহার হয়?

দুটি DataFrame-এর common key বা column ধরে relational join করার জন্য। Inner, left, right ও outer join করা যায়।

### ৩৯. বড় CSV কীভাবে memory-efficientভাবে process করবে?

শুধু প্রয়োজনীয় column load, উপযুক্ত dtype, chunksize দিয়ে batch processing, vectorized operation এবং প্রয়োজনে Parquet বা database ব্যবহার করব।

### ৪০. `apply(axis=1)` সবসময় ভালো পদ্ধতি নয় কেন?

এটি row-by-row Python function call করতে পারে, যা vectorized operation-এর তুলনায় ধীর। Built-in vectorized operation সম্ভব হলে সেটি পছন্দ করব।

## Scenario-based Questions

### ৪১. একটি ১০ GB CSV file process করতে হলে কী করবে?

পুরো file DataFrame-এ load না করে chunksize দিয়ে batch process করব। প্রয়োজনীয় column-ই load করব, dtype optimize করব এবং partial aggregate জমা করব। প্রয়োজন হলে DuckDB, Polars বা distributed processing বিবেচনা করব।

### ৪২. API থেকে পাওয়া JSON data কীভাবে নিরাপদে process করবে?

প্রথমে Pydantic বা সমমানের schema দিয়ে structure ও constraints validate করব। তারপর authentication/authorization এবং domain-specific business rule enforce করব। Validation failure log করব, কিন্তু sensitive payload অযথা log করব না।

### ৪৩. Async API call কখন ব্যবহার করবে?

একাধিক independent I/O-bound request থাকলে এবং ব্যবহৃত HTTP/database client async support করলে। Timeout, retry, cancellation, rate limit ও bounded concurrency রাখব।

### ৪৪. NumPy operation-এর memory usage হঠাৎ বেড়ে গেলে কী করবে?

Array-এর dtype, shape ও intermediate copy inspect করব। প্রয়োজন ছাড়া copy/reshape এড়াব, উপযুক্ত dtype ব্যবহার করব এবং chunked processing বিবেচনা করব।

### ৪৫. DataFrame-এ duplicate transaction থাকলে কী করবে?

প্রথমে transaction-এর unique identifier ও duplicate definition নির্ধারণ করব। তারপর `duplicated(subset=...)` দিয়ে inspect করে business rule অনুযায়ী keep/drop করব। Blindly সব duplicate row drop করব না।

## নিজের উত্তর যাচাই

প্রতিটি প্রশ্নের উত্তর:
- ১ মিনিটের মধ্যে পরিষ্কারভাবে বলতে পারি কি?
- একটি practical example দিতে পারি কি?
- কখন ব্যবহার করব এবং limitation কী তা বলতে পারি কি?
- প্রয়োজন হলে ছোট code লিখতে পারি কি?

শুধু definition মুখস্থ না করে trade-off ও real-world use case ব্যাখ্যা করার অনুশীলন করো।
