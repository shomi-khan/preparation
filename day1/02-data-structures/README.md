# Python Data Structures

একটি data structure শুধু data রাখার পাত্র নয়। এটি ঠিক করে দেয় data কীভাবে access, update, search এবং organize হবে। ভুল structure বেছে নিলে code ধীর হতে পারে, অপ্রয়োজনীয় memory খরচ হতে পারে, এমনকি subtle bug-ও তৈরি হতে পারে।

এই chapter-এ Python-এর built-in collections এবং interview-এ বেশি আসা practical structures নিয়ে আলোচনা করা হয়েছে।

## ১. Python-এর built-in collection-গুলো কী এবং কখন ব্যবহার করব?

Python-এর প্রধান built-in collection চারটি:

| Structure | Ordered? | Mutable? | Duplicate? | Typical use |
|---|---|---|---|---|
| `list` | হ্যাঁ | হ্যাঁ | হ্যাঁ | পরিবর্তনযোগ্য sequence |
| `tuple` | হ্যাঁ | না, item binding | হ্যাঁ | fixed record / sequence |
| `set` | insertion order guarantee নয় | হ্যাঁ | না | uniqueness ও membership |
| `dict` | হ্যাঁ, insertion order | হ্যাঁ | key unique | key-value mapping |

কোনটি বেছে নেবে তা operation-এর উপর নির্ভর করে:

~~~mermaid
flowchart TD
    A["কী ধরনের কাজ করতে হবে?"] --> B{"Key দিয়ে value খুঁজবে?"}
    B -->|হ্যাঁ| D["dict"]
    B -->|না| C{"Unique item দরকার?"}
    C -->|হ্যাঁ| E["set"]
    C -->|না| F{"Sequence পরিবর্তন হবে না?"}
    F -->|হ্যাঁ| G["tuple"]
    F -->|না| H["list"]
~~~

## ২. List কীভাবে কাজ করে?

List হলো ordered, mutable sequence। Duplicate এবং different type-এর object রাখা যায়।

~~~python
items = ["python", "numpy", "pandas"]
items.append("pytorch")
items[0] = "Python"

print(items)
# ['Python', 'numpy', 'pandas', 'pytorch']
~~~

CPython-এ list মূলত dynamic array-এর মতো: element-গুলোর reference contiguous array-তে রাখা হয়। তাই index দিয়ে access দ্রুত হয়। Capacity শেষ হলে বড় storage allocate করে reference copy করতে হতে পারে।

~~~mermaid
flowchart LR
    A["items"] --> B["List object"]
    B --> C["0: ref → 'python'"]
    B --> D["1: ref → 'numpy'"]
    B --> E["2: ref → 'pandas'"]
~~~

### Common list operation-এর complexity

| Operation | Typical complexity | কারণ |
|---|---:|---|
| `items[i]` | O(1) | index থেকে সরাসরি slot access |
| `append(x)` | amortized O(1) | মাঝে মাঝে resize লাগে |
| `pop()` | O(1) | শেষ item সরানো |
| `insert(0, x)` | O(n) | বাকি reference shift হয় |
| `pop(0)` | O(n) | বাকি reference shift হয় |
| `x in items` | O(n) | একে একে compare করতে হয় |
| `items.sort()` | O(n log n) | comparison sort |

**Interview point:** `append()`-কে amortized O(1) বলা হয়, কারণ প্রতিটি append আলাদা করে O(1) না হলেও অনেকগুলো append-এর মোট খরচ গড়ে প্রতি operation-এ constant থাকে।

## ৩. Tuple কী? Tuple কি পুরোপুরি immutable?

Tuple হলো ordered sequence, যার item reference বা binding পরিবর্তন করা যায় না।

~~~python
point = (10, 20)
x, y = point

# point[0] = 99  # TypeError
~~~

কিন্তু tuple-এর ভেতরে mutable object থাকলে সেই object-এর state পরিবর্তন করা যায়:

~~~python
data = ([1, 2], "fixed")
data[0].append(3)

print(data)  # ([1, 2, 3], 'fixed')
~~~

~~~mermaid
flowchart TD
    A["Tuple object"] --> B["Slot 0"]
    A --> C["Slot 1"]
    B --> D["Mutable list [1, 2]"]
    C --> E["String 'fixed'"]
    D --> F["append(3) allowed"]
    A -. "Slot 0-কে অন্য object-এ rebind করা যাবে না" .-> G["Immutable tuple structure"]
~~~

Tuple ব্যবহার করো যখন একসঙ্গে থাকা values-এর সংখ্যা/অর্থ স্থির, যেমন coordinate বা database row-এর মতো record।

## ৪. Set কীভাবে কাজ করে?

Set unique, hashable element রাখে। Duplicate insert করলে একই value দ্বিতীয়বার যোগ হয় না।

~~~python
tags = {"ai", "python", "ml", "ai"}
print(tags)  # 'ai' একবার থাকবে

a = {1, 2, 3}
b = {3, 4, 5}

print(a | b)  # union: {1, 2, 3, 4, 5}
print(a & b)  # intersection: {3}
print(a - b)  # difference: {1, 2}
print(a ^ b)  # symmetric difference: {1, 2, 4, 5}
~~~

Set membership সাধারণত average O(1), কারণ set hash table ব্যবহার করে। Set iteration order-কে application contract হিসেবে ধরে নেওয়া উচিত নয়।

~~~mermaid
flowchart TD
    A["Add item to set"] --> B["hash(item)"]
    B --> C["Find candidate slot"]
    C --> D{"Equivalent item exists?"}
    D -->|হ্যাঁ| E["Duplicate: set unchanged"]
    D -->|না| F["Insert new item"]
~~~

## ৫. Dictionary কীভাবে কাজ করে?

Dictionary হলো key-value mapping। Key hashable হতে হবে এবং key unique। Python 3.7+ থেকে insertion order language specification-এর অংশ।

~~~python
user = {"id": 7, "name": "Shomi"}
user["role"] = "engineer"

print(user["name"])                 # Shomi
print(user.get("email"))            # None
print(user.get("email", "unknown")) # unknown
~~~

Dictionary lookup সাধারণত average O(1)। Python hash table ব্যবহার করে key-এর hash থেকে সম্ভাব্য location খুঁজে নেয়, তারপর equality check করে সঠিক key নিশ্চিত করে।

~~~mermaid
flowchart TD
    A["Lookup key"] --> B["hash(key)"]
    B --> C["Candidate slot(s)"]
    C --> D["Compare stored key using equality"]
    D --> E{"Matching key found?"}
    E -->|হ্যাঁ| F["Return associated value"]
    E -->|না| G["KeyError or default"]
~~~

## ৬. Hash collision কী?

দুটি ভিন্ন key-এর hash একই হতে পারে। একে hash collision বলে। Hash table collision সামলায়; তাই hash একই হলেই key একই, এমন নয়।

~~~mermaid
flowchart TD
    A["Key A"] --> H["Same hash value"]
    B["Key B"] --> H
    H --> C["Candidate bucket / probe sequence"]
    C --> D["Equality check"]
    D --> E["A == B?"]
    E -->|হ্যাঁ| F["Same logical key"]
    E -->|না| G["Different keys: collision handled"]
~~~

**Interview point:** Hash lookup average O(1), কিন্তু worst-case O(n) হতে পারে। ভালো hash distribution এবং collision handling performance-এ গুরুত্বপূর্ণ।

## ৭. Dictionary key বা set element hashable হতে হয় কেন?

Hash table key-কে নির্দিষ্ট location-এ খুঁজতে hash ব্যবহার করে। তাই key-এর hash dictionary-তে থাকা অবস্থায় স্থিতিশীল থাকতে হবে। সাধারণত immutable built-in values hashable, কিন্তু immutable হলেই hashable হবে এমন নয়।

~~~python
valid = {
    "name": "Shomi",
    10: "ten",
    (1, 2): "coordinate",
}

# invalid = {[1, 2]: "list"}  # TypeError: list unhashable
~~~

Tuple তখনই hashable, যখন তার সব element hashable।

| Type | সাধারণত hashable? |
|---|---|
| `int`, `str`, `bytes` | হ্যাঁ |
| `None`, `bool`, `float` | হ্যাঁ |
| `tuple` | সব element hashable হলে |
| `frozenset` | সব element hashable হলে |
| `list`, `dict`, `set` | না |

Custom class-এ `__eq__` implement করলে hash behavior-ও বুঝে implement করতে হবে। Equality-তে অংশ নেওয়া mutable state বদলে hash বদলে গেলে dictionary/set-এর lookup ভেঙে যেতে পারে।

## ৮. List বনাম Set: membership check-এর জন্য কোনটি ভালো?

~~~python
allowed_ids = [101, 102, 103, 104]
allowed_id_set = set(allowed_ids)

print(103 in allowed_ids)      # O(n)
print(103 in allowed_id_set)   # average O(1)
~~~

একবার বা অল্প কয়েকবার search করলে list-ই যথেষ্ট হতে পারে। একই collection-এ বারবার membership check করলে set ভালো হতে পারে। তবে set তৈরির initial cost এবং অতিরিক্ত memory বিবেচনা করতে হবে।

## ৯. Shallow copy আর shared reference কি একই জিনিস?

না। Assignment নতুন collection তৈরি করে না; shallow copy নতুন outer collection তৈরি করে, কিন্তু nested object-এর reference share করে।

~~~python
original = {"roles": ["reader"]}
alias = original
shallow = original.copy()

alias["roles"].append("writer")
shallow["roles"].append("admin")

print(original)  # {'roles': ['reader', 'writer', 'admin']}
print(alias is original)    # True
print(shallow is original)  # False
print(shallow["roles"] is original["roles"])  # True
~~~

~~~mermaid
flowchart LR
    O["original"] --> OD["Outer dict A"]
    A["alias"] --> OD
    S["shallow"] --> SD["Outer dict B"]
    OD --> L["Shared nested list"]
    SD --> L
~~~

Nested data-ও independent করতে হলে `copy.deepcopy()` ব্যবহার করা যায়। তবে deep copy সব object-এর জন্য উপযুক্ত নয়; custom object, file handle, lock বা external resource copy করার semantics আলাদা হতে পারে।

## ১০. Stack কী? Python-এ কীভাবে implement করব?

Stack হলো LIFO (Last In, First Out): সর্বশেষে ঢোকা item আগে বের হয়। List-এর `append()` এবং `pop()` দিয়ে সহজে implement করা যায়।

~~~python
stack = []
stack.append("first")
stack.append("second")

print(stack.pop())  # second
print(stack.pop())  # first
~~~

~~~mermaid
flowchart TD
    A["push('first')"] --> B["[first]"]
    B --> C["push('second')"]
    C --> D["[first, second]"]
    D --> E["pop()"]
    E --> F["Returns 'second'"]
~~~

Stack ব্যবহার হয় function call stack, undo history, expression parsing এবং depth-first search-এর মতো কাজে।

## ১১. Queue কী? List-এর বদলে deque কেন?

Queue হলো FIFO (First In, First Out): আগে ঢোকা item আগে বের হয়। Python-এ `collections.deque` queue-এর জন্য উপযোগী।

~~~python
from collections import deque

queue = deque()
queue.append("first")
queue.append("second")

print(queue.popleft())  # first
print(queue.popleft())  # second
~~~

List-এর `pop(0)` O(n), কারণ বাকি item shift করতে হয়। Deque-এর দুই প্রান্তে append/pop সাধারণত O(1)।

~~~mermaid
flowchart LR
    A["Enqueue: first"] --> B["[first]"]
    B --> C["Enqueue: second"]
    C --> D["[first, second]"]
    D --> E["Dequeue / popleft()"]
    E --> F["Returns first"]
~~~

## ১২. Counter কী কাজে লাগে?

`collections.Counter` হলো frequency counting-এর জন্য specialized dictionary।

~~~python
from collections import Counter

words = ["ai", "ml", "ai", "python", "ml", "ai"]
counts = Counter(words)

print(counts["ai"])           # 3
print(counts.most_common(2))  # [('ai', 3), ('ml', 2)]
~~~

Frequency count, most common item এবং multiset-style operation-এ Counter সুবিধাজনক।

## ১৩. defaultdict কখন ব্যবহার করব?

কোনো key প্রথমবার এলে default value স্বয়ংক্রিয়ভাবে তৈরি করতে `defaultdict` ব্যবহার করা যায়।

~~~python
from collections import defaultdict

groups = defaultdict(list)
records = [("engineering", "A"), ("design", "B"), ("engineering", "C")]

for department, person in records:
    groups[department].append(person)

print(dict(groups))
# {'engineering': ['A', 'C'], 'design': ['B']}
~~~

Frequency counting-এর জন্য `Counter` বেশি expressive হতে পারে; grouping-এর জন্য `defaultdict(list)` সুবিধাজনক।

## ১৪. Heap কী? কখন ব্যবহার করব?

Heap এমন একটি structure যা দ্রুত smallest বা largest-priority item বের করতে সাহায্য করে। Python-এর `heapq` min-heap দেয়। Root-এ সবচেয়ে ছোট item থাকে।

~~~python
import heapq

numbers = [7, 2, 9, 1, 5]
heapq.heapify(numbers)

print(heapq.heappop(numbers))  # 1
heapq.heappush(numbers, 3)
print(heapq.heappop(numbers))  # 2
~~~

~~~mermaid
flowchart TD
    A["Min-heap"] --> B["Root = smallest item"]
    B --> C["heappop() returns root"]
    C --> D["Heap property restored"]
    E["heappush(new item)"] --> D
~~~

Typical complexity: `heapify()` O(n), `heappush()` O(log n), `heappop()` O(log n), smallest item peek O(1)। Top-K problem, priority scheduling এবং merge-sorted-streams-এ heap কাজে লাগে।

## ১৫. List comprehension কখন ব্যবহার করব?

একটি iterable থেকে নতুন list তৈরি করার concise syntax হলো list comprehension।

~~~python
squares = [number * number for number in range(5)]
evens = [number for number in range(10) if number % 2 == 0]
~~~

শুধু প্রয়োজনমতো একটির পর একটি value process করতে চাইলে generator expression বেশি memory-efficient হতে পারে। কিন্তু result বারবার ব্যবহার বা index করতে হলে list সুবিধাজনক।

## ১৬. Sorting-এর সময় key function কেন ব্যবহার করব?

`sorted()` নতুন sorted list দেয়; `list.sort()` existing list-কে in-place sort করে। `key` দিয়ে sorting criterion নির্দিষ্ট করা যায়।

~~~python
users = [
    {"name": "Rafi", "age": 30},
    {"name": "Mina", "age": 25},
    {"name": "Asha", "age": 28},
]

by_age = sorted(users, key=lambda user: user["age"])
print([user["name"] for user in by_age])
# ['Mina', 'Asha', 'Rafi']
~~~

Python sorting stable: equal key-র item-গুলোর relative order আগের মতো থাকে। সাধারণ comparison sort complexity O(n log n)।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| List index access? | O(1) |
| List-এর মাঝখানে insert? | O(n) |
| List append? | Amortized O(1) |
| Set membership? | Average O(1) |
| Dict lookup? | Average O(1), worst-case O(n) |
| Set duplicate রাখে? | না |
| Tuple পুরোপুরি immutable? | Tuple-এর item binding immutable; nested mutable object বদলাতে পারে |
| Dict key কী হতে হবে? | Hashable |
| Stack কোন order? | LIFO |
| Queue কোন order? | FIFO |
| Python queue-এর জন্য? | `collections.deque` |
| Top-K / priority queue? | `heapq` |
| Frequency count? | `collections.Counter` |
| Grouping? | `defaultdict` |

## Practice Exercises

1. একটি list থেকে duplicate বাদ দিয়ে original order বজায় রাখো। Input: `[3, 1, 3, 2, 1]`; expected: `[3, 1, 2]`।
2. দুটি list-এর common unique element বের করো।
3. একটি sentence-এর প্রতিটি word-এর frequency গণনা করো।
4. `deque` দিয়ে queue implement করো। Empty queue থেকে dequeue করলে কী হবে, সেই behavior define করে test লেখো।
5. List of dictionaries থেকে নির্দিষ্ট status-এর record filter করো।
6. Nested list-সহ dictionary-তে assignment, shallow copy ও deep copy-এর mutation behavior compare করো।
7. `set` ও `frozenset` compare করো এবং কখন কোনটি বেছে নেবে ব্যাখ্যা করো।
8. `heapq` দিয়ে একটি list থেকে তিনটি smallest number বের করো।
9. Dictionary-এর value অনুযায়ী records sort করো।
10. একই workload-এর জন্য list membership এবং set membership compare করো। Collection তৈরির cost-ও বিবেচনা করো।

## Practice করার পদ্ধতি

1. Code লেখার আগে expected output predict করো।
2. প্রতিটি example নিজে run করে output ব্যাখ্যা করো।
3. Empty input, duplicate value, missing key এবং nested mutable object-এর মতো edge case পরীক্ষা করো।
4. প্রতিটি operation-এর time complexity কেন এমন, তা নিজের ভাষায় ব্যাখ্যা করো।
5. শুধু syntax নয়, কেন একটি structure অন্যটির চেয়ে উপযুক্ত সেটিও justify করো।
