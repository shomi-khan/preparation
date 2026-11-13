# Python Data Structures

## ১. List কী?

List হলো ordered, mutable collection। Duplicate value রাখা যায়।

```python
items = ["python", "numpy", "pandas"]
items.append("pytorch")
items[0] = "Python"
```

ব্যবহার: sequence, ordered records, পরিবর্তনশীল collection।

সাধারণ complexity:
- Index access: O(1)
- Append (amortized): O(1)
- মাঝখানে insert/delete: O(n)
- Value search: O(n)

## ২. Tuple কী?

Tuple হলো ordered, immutable collection।

```python
point = (10, 20)
x, y = point  # unpacking
```

ব্যবহার: fixed-size record, function থেকে একাধিক value return, hashable key (সব element hashable হলে)।

## ৩. Set কী?

Set হলো unique element-এর unordered collection। Membership test গড়ে O(1)।

```python
tags = {"ai", "python", "ml", "ai"}
print(tags)  # duplicate "ai" থাকবে একবার

print("python" in tags)

a = {1, 2, 3}
b = {3, 4, 5}
print(a | b)  # union
print(a & b)  # intersection
print(a - b)  # difference
```

ব্যবহার: duplicate বাদ দেওয়া, membership check, set operations।

## ৪. Dictionary কী?

Dictionary key-value mapping। Key hashable হতে হয়। Python 3.7+ এ insertion order language guarantee।

```python
user = {"id": 7, "name": "Shomi"}
user["role"] = "engineer"

print(user.get("email"))  # key না থাকলে None
print(user.get("email", "not-provided"))
```

ব্যবহার: lookup, configuration, JSON-like data।

সাধারণত key lookup, insert ও delete গড়ে O(1), তবে worst-case complexity বেশি হতে পারে।

## ৫. কোন collection কখন ব্যবহার করবে?

| প্রয়োজন | উপযুক্ত structure |
|---|---|
| Ordered, পরিবর্তনযোগ্য sequence | list |
| Ordered, অপরিবর্তনীয় sequence | tuple |
| Unique values / membership | set |
| Key-value lookup | dict |

## ৬. List বনাম Set membership

```python
allowed_ids = [101, 102, 103, 104]
allowed_set = set(allowed_ids)

print(103 in allowed_ids)  # O(n)
print(103 in allowed_set)  # average O(1)
```

অনেক membership check করতে হলে set-এ convert করা লাভজনক হতে পারে। তবে set তৈরির initial cost ও memory বিবেচনা করতে হবে।

## ৭. Stack এবং Queue কীভাবে implement করবে?

Stack: Last In, First Out (LIFO)। List-এর `append()` ও `pop()` ব্যবহার করা যায়।

Queue: First In, First Out (FIFO)। `collections.deque` ব্যবহার করা ভালো; list-এর শুরু থেকে `pop(0)` করলে O(n) সময় লাগে।

```python
from collections import deque

stack = []
stack.append("first")
stack.append("second")
print(stack.pop())  # second

queue = deque()
queue.append("first")
queue.append("second")
print(queue.popleft())  # first
```

## ৮. Dictionary দিয়ে frequency count

```python
from collections import Counter

words = ["ai", "ml", "ai", "python", "ml", "ai"]
counts = Counter(words)
print(counts)
print(counts.most_common(2))
```

## Practice

1. একটি list থেকে duplicate বাদ দিয়ে original order বজায় রাখো।
2. দুটি list-এর common element বের করো।
3. একটি sentence-এর প্রতিটি word কতবার এসেছে তা গণনা করো।
4. List ব্যবহার না করে `deque` দিয়ে queue তৈরি করো।
5. একটি list of dictionaries থেকে নির্দিষ্ট status-এর record filter করো।
