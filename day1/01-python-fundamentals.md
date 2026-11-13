# Python Fundamentals

## ১. Python কী?

Python একটি high-level, general-purpose programming language। এর syntax তুলনামূলক সহজ এবং data processing, automation, web development, Machine Learning ও AI application development-এ ব্যাপকভাবে ব্যবহৃত হয়।

Python code সাধারণত CPython interpreter দিয়ে execute হয়। CPython source code-কে bytecode-এ compile করে এবং Python Virtual Machine সেই bytecode execute করে।

## ২. Mutable এবং Immutable কী?

**Mutable object** তৈরি হওয়ার পর তার internal state পরিবর্তন করা যায়। যেমন: `list`, `dict`, `set`।

**Immutable object** তৈরি হওয়ার পর তার value পরিবর্তন করা যায় না। যেমন: `int`, `float`, `bool`, `str`, `tuple` (যদিও tuple-এর ভেতরে mutable object থাকলে সেই object পরিবর্তনযোগ্য হতে পারে)।

```python
numbers = [1, 2, 3]
numbers.append(4)  # একই list পরিবর্তিত হলো

name = "Shomi"
name = name + " Khan"  # নতুন string তৈরি হলো
```

Immutable object hashable হতে পারে, তাই dictionary key হিসেবে ব্যবহার করা যায়। তবে সব immutable container স্বয়ংক্রিয়ভাবে hashable নয়: tuple-এর সব element-ও hashable হতে হবে।

## ৩. `is` এবং `==`-এর পার্থক্য কী?

- `==` value equality পরীক্ষা করে।
- `is` object identity পরীক্ষা করে, অর্থাৎ দুটি reference একই object নির্দেশ করছে কি না।

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True: value একই
print(a is b)  # False: object আলাদা
```

সাধারণত `None` যাচাই করতে `value is None` লেখা হয়।

## ৪. Shallow Copy এবং Deep Copy কী?

Shallow copy outer container-এর নতুন object তৈরি করে, কিন্তু nested object-গুলোর reference একই রাখে। Deep copy nested object-সহ recursive copy তৈরি করে।

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
deep = copy.deepcopy(original)

shallow[0].append(99)
print(original)  # [[1, 2, 99], [3, 4]]

deep[1].append(88)
print(original)  # deep copy-এর পরিবর্তন original-এ আসবে না
```

Deep copy ব্যয়বহুল হতে পারে এবং সব object-এর জন্য উপযুক্তও নয়।

## ৫. Scope এবং LEGB Rule কী?

Python variable lookup-এর ক্ষেত্রে LEGB rule অনুসরণ করে:

1. **Local:** বর্তমান function scope।
2. **Enclosing:** outer বা enclosing function scope।
3. **Global:** module-level scope।
4. **Built-in:** Python-এর built-in names।

```python
message = "global"

def outer():
    message = "enclosing"

    def inner():
        message = "local"
        print(message)

    inner()

outer()  # local
```

## ৬. `*args` এবং `**kwargs` কী?

- `*args` অতিরিক্ত positional arguments-কে tuple হিসেবে গ্রহণ করে।
- `**kwargs` অতিরিক্ত keyword arguments-কে dictionary হিসেবে গ্রহণ করে।

```python
def report(title, *values, **metadata):
    print(title)
    print(values)
    print(metadata)

report("Sales", 10, 20, region="Asia", year=2026)
```

## ৭. Comprehension কী?

Comprehension দিয়ে iterable থেকে সংক্ষিপ্তভাবে list, set বা dictionary তৈরি করা যায়।

```python
squares = [n * n for n in range(5)]
unique_lengths = {len(word) for word in ["ai", "python", "ml"]}
length_by_word = {word: len(word) for word in ["ai", "python"]}
```

বড় বা জটিল transformation হলে readability-এর জন্য সাধারণ loop ভালো হতে পারে।

## ৮. Python-এ সবকিছু কি object?

Python-এ integer, function, class, module-সহ প্রায় সব runtime value object। Function-কে variable-এ রাখা, argument হিসেবে পাঠানো এবং return করা যায়। এ কারণেই Python-এ higher-order function ও decorator সম্ভব।

```python
def greet(name):
    return f"Hello, {name}"

say_hello = greet
print(say_hello("World"))
```

## Interview-এ মনে রাখবে

- Python dynamically typed, কিন্তু type hints দিয়ে static analysis করা যায়।
- Assignment সাধারণত object copy করে না; reference bind করে।
- Mutable default argument এড়িয়ে চলবে।
- `is` identity, `==` equality।
- Readability এবং explicitness Python code-এর গুরুত্বপূর্ণ গুণ।

## Mutable default argument-এর সমস্যা

```python
# ভুল pattern: একই list call-গুলোর মধ্যে reuse হয়
def add_item(item, items=[]):
    items.append(item)
    return items

# নিরাপদ pattern
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```
