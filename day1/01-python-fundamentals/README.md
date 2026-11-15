# Python Fundamentals: Interview Questions & Answers

এই অধ্যায়ের লক্ষ্য Python-এর core concepts পরিষ্কারভাবে বোঝা, code দেখে behavior predict করা এবং interview-তে নিজের সিদ্ধান্তের কারণ ব্যাখ্যা করতে পারা। এখানে প্রতিটি প্রশ্নের সঙ্গেই উত্তর, runnable example এবং প্রয়োজনীয় visual দেওয়া হয়েছে। আলাদা Visual Guide section নেই।

> **কীভাবে পড়বে:** প্রশ্নটি আগে নিজে উত্তর দেওয়ার চেষ্টা করো। তারপর explanation পড়ো, code run করো এবং output কেন এমন হলো নিজের ভাষায় ব্যাখ্যা করো।

---

## 1. Python কী? এটি dynamically typed language কেন?

Python একটি high-level, general-purpose programming language। এটি dynamically typed, অর্থাৎ variable declare করার সময় type লিখে দেওয়া বাধ্যতামূলক নয়। তবে প্রতিটি object-এর type থাকে।

~~~python
value = 100
print(type(value))  # <class 'int'>

value = "one hundred"
print(type(value))  # <class 'str'>
~~~

একই name প্রথমে integer object, পরে string object-কে refer করছে। Integer object string-এ mutate হয়নি।

**মনে রাখবে:** Dynamic typing মানে type নেই, তা নয়। Object-এর type runtime-এ থাকে।

## 2. Python কি compiled, নাকি interpreted language?

সাধারণভাবে Python-কে interpreted language বলা হয়। প্রচলিত CPython implementation source code-কে bytecode-এ compile করে, তারপর Python runtime সেই bytecode execute করে।

~~~text
Python source (.py)
        |
        v
     Bytecode
        |
        v
  Python runtime
        |
        v
      Result
~~~

Bytecode CPU-র native machine code নয়। Interview-তে সাধারণত এতটুকু বোঝাতে পারলেই যথেষ্ট। Python কীভাবে run হয় তার প্রতিটি internal ধাপ মুখস্থ করার প্রয়োজন নেই।

## 3. CPython কী?

CPython হলো Python language-এর সবচেয়ে প্রচলিত implementation এবং reference implementation। এটি মূলত C programming language দিয়ে implement করা হয়েছে। Python language আর CPython এক জিনিস নয়।

~~~python
import platform
print(platform.python_implementation())
~~~

সাধারণ installation-এ output হবে **CPython**। অন্য implementation-এর উদাহরণ PyPy, যেখানে JIT compilation রয়েছে।

## 4. Python-এ variable কী? Variable কি box?

Python-এ variable-কে box হিসেবে ভাবলে ভুল ধারণা তৈরি হতে পারে। ভালো mental model হলো: **variable একটি name, যা কোনো object-কে refer করে।**

~~~python
first = [10, 20]
second = first
print(first is second)  # True
~~~

~~~text
first  -----+
            +----> List object [10, 20]
second -----+
~~~

দুটি name, কিন্তু list object একটি। **second = first** নতুন list তৈরি করেনি।

## 5. Assignment করলে কি object copy হয়?

না। সাধারণ assignment একটি name-কে object-এর সঙ্গে bind করে। এটি object copy করে না।

~~~python
a = [1, 2]
b = a
b.append(3)

print(a)  # [1, 2, 3]
print(b)  # [1, 2, 3]
~~~

~~~text
a -----+
       +----> [1, 2, 3]
b -----+
~~~

দুইটি name একই list-কে refer করছে। তাই একটি name দিয়ে list mutate করলে অন্য name দিয়েও পরিবর্তনটি দেখা যায়।

## 6. Mutable এবং Immutable object কী?

**Mutable object** তৈরি হওয়ার পরেও যার state পরিবর্তন করা যায়। **Immutable object** তৈরি হওয়ার পর যার state পরিবর্তন করা যায় না।

| Mutable examples | Immutable examples |
|---|---|
| list | int |
| dict | float |
| set | bool |
| অধিকাংশ custom class instance | str, bytes |
| | tuple (শর্তসাপেক্ষে hashable) |
| | frozenset |

~~~python
items = [1, 2]
before = id(items)
items.append(3)
print(items)                # [1, 2, 3]
print(id(items) == before)  # True

name = "Shomi"
name = name + " Khan"
print(name)                 # Shomi Khan
~~~

List-এর state একই object-এ পরিবর্তিত হয়েছে। String mutate হয়নি, নতুন string তৈরি হয়েছে এবং name নতুন object-কে refer করছে।

**Immutable object মানে variable reassign করা যাবে না, এমন নয়।** Immutable হলো object-এর property, variable name-এর নয়।

## 7. একই object-এর content বদলালে identity কি বদলায়?

না। Mutation হলে object-এর state বদলায়, কিন্তু identity একই থাকতে পারে।

~~~python
numbers = [10, 20]
original_id = id(numbers)
numbers.append(30)

print(numbers)  # [10, 20, 30]
print(id(numbers) == original_id)  # True
~~~

~~~text
আগে: numbers ---> List A [10, 20]
                         |
                      append(30)
                         |
পরে: numbers ---> List A [10, 20, 30]

Object একই, state বদলেছে।
~~~

অন্যদিকে name-কে নতুন list assign করলে name নতুন object-এ rebind হবে।

## 8. is এবং ==-এর পার্থক্য কী?

- **==** দুই object-এর value/content সমান কি না পরীক্ষা করে।
- **is** দুই name একই object-কে refer করছে কি না পরীক্ষা করে।

~~~python
a = [1, 2]
b = [1, 2]
c = a

print(a == b)  # True
print(a is b)  # False
print(a is c)  # True
~~~

~~~text
a ----> List A [1, 2] <---- c
b ----> List B [1, 2]

a == b : content একই
a is b : object একই নয়
a is c : object একই
~~~

**None** যাচাইয়ে সাধারণত **is None** ব্যবহার করা হয়। Integer বা string-এর value compare করতে **is** ব্যবহার করবে না।

## 9. Shallow Copy কী?

Shallow copy নতুন **outer object** তৈরি করে, কিন্তু nested object-গুলোকে copy করে না। Nested object-এর reference share হয়।

~~~python
import copy

original = {
    "name": "Python",
    "versions": [3, 11, 12]
}
shallow = copy.copy(original)

print(original is shallow)  # False
print(original["versions"] is shallow["versions"])  # True
~~~

~~~text
original dict ---> versions ---+
                               +---> একই nested list [3, 11, 12]
shallow dict  ---> versions ---+

Outer dictionary আলাদা।
ভেতরের list একই।
~~~

Nested list পরিবর্তন করলে:

~~~python
shallow["versions"].append(13)
print(original["versions"])  # [3, 11, 12, 13]
print(shallow["versions"])   # [3, 11, 12, 13]
~~~

Original-ও বদলেছে, কারণ দুই dictionary-র versions key একই list object-কে refer করছে।

তবে outer dictionary-তে নতুন key যোগ করলে original-এ তা যোগ হবে না:

~~~python
shallow["language"] = "Python"
print("language" in original)  # False
~~~

কারণ outer dictionary দুটি আলাদা object।

## 10. Deep Copy কী?

Deep copy নতুন outer object তৈরি করে এবং nested object-গুলোও recursively copy করার চেষ্টা করে।

~~~python
import copy

original = {
    "name": "Python",
    "versions": [3, 11, 12]
}
deep = copy.deepcopy(original)

print(original is deep)  # False
print(original["versions"] is deep["versions"])  # False

deep["versions"].append(13)
print(original["versions"])  # [3, 11, 12]
print(deep["versions"])      # [3, 11, 12, 13]
~~~

~~~text
original dict ---> original list [3, 11, 12]

deep dict     ---> copied list   [3, 11, 12]

Outer dictionary আলাদা।
Nested list-ও আলাদা।
~~~

### Shallow Copy বনাম Deep Copy

| বিষয় | Shallow Copy | Deep Copy |
|---|---|---|
| Outer object | নতুন | নতুন |
| Nested object | সাধারণত reference share করে | recursively copy করার চেষ্টা করে |
| Nested list mutate করলে original বদলাতে পারে? | হ্যাঁ | সাধারণত না |
| Cost | তুলনামূলক কম | তুলনামূলক বেশি |
| Function | copy.copy() | copy.deepcopy() |

**কখন কোনটি?**
- শুধু outer container আলাদা দরকার, nested data share করা গ্রহণযোগ্য: shallow copy।
- Nested mutable data-ও আলাদা দরকার: deep copy।
- অন্ধভাবে deep copy করবে না। এটি বেশি memory/time নিতে পারে। File, socket, database connection-এর মতো resource meaningfulভাবে copy করা যায় না। Custom object-এর copy behavior-ও আলাদা হতে পারে।

**Interview answer:** Shallow copy outer object copy করে, কিন্তু nested object-এর reference share করে। Deep copy nested structure-সহ recursively copy করার চেষ্টা করে।

## 11. Function-এ argument কীভাবে pass হয়?

Python-এ argument passing-কে **call by object sharing** বলা হয়। Function parameter একটি নতুন local name, যা caller-এর দেওয়া object-কে refer করে।

~~~python
def add_item(items):
    items.append("new")

data = ["old"]
add_item(data)
print(data)  # ['old', 'new']
~~~

~~~text
caller: data -------+
                    +----> একই list object
function: items ----+          |
                              append()
                                 |
                          ['old', 'new']
~~~

কিন্তু function-এর parameter-কে নতুন object assign করলে caller-এর name rebind হয় না:

~~~python
def replace_items(items):
    items = ["replacement"]

data = ["original"]
replace_items(data)
print(data)  # ['original']
~~~

Function শুধু নিজের local name-কে নতুন list-এ bind করেছে।

## 12. Mutable default argument-এর সমস্যা কী?

Function definition execute হওয়ার সময় default argument একবার evaluate হয়। তাই default হিসেবে list বা dictionary দিলে পরবর্তী call-গুলো একই object reuse করতে পারে।

~~~python
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item("A"))  # ['A']
print(add_item("B"))  # ['A', 'B']
~~~

~~~text
Function definition
       |
       v
একটি default list তৈরি: []
       |
       +---- Call 1: append("A") -> ["A"]
       |
       +---- Call 2: append("B") -> ["A", "B"]

দুই call-এ একই default list।
~~~

সঠিক pattern:

~~~python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

print(add_item("A"))  # ['A']
print(add_item("B"))  # ['B']
~~~

এখানে প্রতিটি call-এ নতুন list তৈরি হয়, যদি caller নিজে list না দেয়।

## 13. List, Tuple, Set ও Dictionary-এর পার্থক্য কী?

| Type | Ordered? | Mutable? | Duplicate | Access |
|---|---|---|---|---|
| list | হ্যাঁ | হ্যাঁ | অনুমোদিত | index |
| tuple | হ্যাঁ | না | অনুমোদিত | index |
| set | index order নেই | হ্যাঁ | রাখে না | index নেই |
| dict | insertion order বজায় রাখে | হ্যাঁ | key unique | key |

~~~python
numbers = [1, 2, 2]
coordinates = (10, 20)
unique_numbers = {1, 2, 2}
user = {"name": "Shomi", "age": 32}

print(unique_numbers)  # {1, 2}
~~~

কোনটি ব্যবহার করবে তা নির্ভর করে data কীভাবে access ও modify করতে হবে তার উপর।

## 14. Dictionary কীভাবে কাজ করে? Hash কী?

Dictionary একটি mapping data structure। এটি key-এর hash ব্যবহার করে key-value entry খুঁজে পেতে সাহায্য করে। Hash হলো একটি integer value, যা hashable object থেকে **hash()** function-এর মাধ্যমে পাওয়া যায়।

~~~python
user = {"name": "Shomi"}
print(hash("name"))
print(user["name"])
~~~

~~~text
key: "name"
     |
     v
  hash(key)
     |
     v
Candidate lookup location
     |
     v
Key equality check
     |
     v
Value: "Shomi"
~~~

Hash unique identifier নয়। দুইটি আলাদা object-এর hash একই হতে পারে, এটিকে hash collision বলে। তাই dictionary hash-এর পাশাপাশি key equality-ও পরীক্ষা করে।

## 15. Hashable object কী? List বা dict key হতে পারে না কেন?

একটি object hashable, যদি তার hash পাওয়া যায় এবং equality behavior hash contract মেনে চলে। Hashable object-এর hash তার lifetime-এ স্থিতিশীল থাকতে হয়।

~~~python
print(hash(100))       # Works
print(hash("python"))  # Works

hash([1, 2])           # TypeError
hash({"a": 1})         # TypeError
~~~

List ও dictionary mutable, তাই তাদের content পরিবর্তন হতে পারে। Built-in list ও dict unhashable।

~~~text
Dictionary-তে key রাখার সময়:
key -> hash(key) -> entry lookup

key-এর hash পরে বদলে গেলে:
পুরোনো lookup location আর নতুন hash মেলে না।
~~~

সব immutable object hashable নয়:

~~~python
print(hash((1, 2)))  # Works
hash((1, [2, 3]))    # TypeError
~~~

Tuple নিজে immutable, কিন্তু এর ভেতরে list আছে। তাই tuple-টির hash করা যায় না।

**মনে রাখবে:** list, dict, set unhashable। int, str এবং hashable elements-যুক্ত tuple hashable।

## 16. None কী? None কি mutable বা hashable?

**None** হলো Python-এর **NoneType**-এর একমাত্র instance। এটি সাধারণত কোনো value অনুপস্থিত বোঝাতে ব্যবহার হয়।

~~~python
result = None
if result is None:
    print("No result")

print(type(None))  # <class 'NoneType'>
print(hash(None))  # integer hash
~~~

None immutable এবং hashable। এটি singleton, তাই None যাচাইয়ে **is None** ব্যবহার করা হয়। None থেকে variable-এ অন্য value assign করা None mutate করা নয়, name rebinding।

## 17. List comprehension এবং Generator expression-এর পার্থক্য কী?

List comprehension এক expression-এ নতুন list তৈরি করে। Generator expression প্রয়োজন অনুযায়ী একে একে value দেয়।

~~~python
squares_list = [x * x for x in range(5)]
squares_gen = (x * x for x in range(5))

print(squares_list)       # [0, 1, 4, 9, 16]
print(next(squares_gen))  # 0
print(next(squares_gen))  # 1
~~~

~~~text
List comprehension:
সব value তৈরি -> [0, 1, 4, 9, 16]

Generator expression:
next() -> 0
next() -> 1
next() -> 4
প্রয়োজন অনুযায়ী value
~~~

অনেক data process করার সময় generator memory-efficient হতে পারে। তবে generator একবার consume হয়ে গেলে সাধারণত আবার শুরু থেকে ব্যবহার করা যায় না।

## 18. Iterable, Iterator এবং Generator-এর পার্থক্য কী?

- **Iterable:** যে object থেকে iterator পাওয়া যায়, যেমন list, tuple, string।
- **Iterator:** যে object **next()** দিয়ে একের পর এক value দেয়।
- **Generator:** iterator তৈরির সহজ উপায়; সাধারণত **yield** ব্যবহার করে।

~~~python
numbers = [10, 20, 30]    # iterable
iterator = iter(numbers)  # iterator

print(next(iterator))  # 10
print(next(iterator))  # 20
print(next(iterator))  # 30
~~~

Generator example:

~~~python
def count_up_to(limit):
    number = 1
    while number <= limit:
        yield number
        number += 1

for value in count_up_to(3):
    print(value)
~~~

**yield** value দেয় এবং function-এর execution state সংরক্ষণ করে। পরের next-এ সেখান থেকে execution চলতে থাকে।

## 19. yield এবং return-এর পার্থক্য কী?

**return** function-এর execution শেষ করে value ফেরত দেয়। **yield** generator-এর value দেয়, কিন্তু execution state রেখে সাময়িকভাবে থামে।

~~~python
def normal_function():
    return 1
    return 2

def generator_function():
    yield 1
    yield 2

print(normal_function())          # 1
print(list(generator_function())) # [1, 2]
~~~

একটি function body-তে yield থাকলে সেটি call করার সময় generator object তৈরি করে।

## 20. Decorator কী?

Decorator এমন একটি function, যা অন্য function বা callable-কে input হিসেবে নিয়ে তার behavior wrap বা extend করে। মূল function-এর code না বদলিয়েও অতিরিক্ত behavior যোগ করা যায়।

~~~python
def log_call(func):
    def wrapper():
        print("Function is starting")
        result = func()
        print("Function is finished")
        return result
    return wrapper

@log_call
def greet():
    print("Hello!")

greet()
~~~

Output:

~~~text
Function is starting
Hello!
Function is finished
~~~

**@log_call** মূলত **greet = log_call(greet)**-এর shorthand।

বাস্তবে decorator-এ প্রায়ই **args**, **kwargs** এবং **functools.wraps** ব্যবহার করা হয়:

~~~python
from functools import wraps

def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
~~~

## 21. *args এবং **kwargs কী?

- **\*args** অতিরিক্ত positional arguments-কে tuple হিসেবে গ্রহণ করে।
- **\*\*kwargs** অতিরিক্ত keyword arguments-কে dictionary হিসেবে গ্রহণ করে।

~~~python
def report(title, *args, **kwargs):
    print(title)
    print(args)
    print(kwargs)

report("Sales", 10, 20, region="Asia", year=2026)
~~~

Output:

~~~text
Sales
(10, 20)
{'region': 'Asia', 'year': 2026}
~~~

Function call করার সময়ও unpacking করা যায়:

~~~python
values = [10, 20]
options = {"sep": " | "}
print(*values, **options)  # 10 | 20
~~~

## 22. Scope কী? LEGB rule কী?

Scope হলো code-এর কোন অংশ থেকে একটি name access করা যায় তার পরিসর। Python name lookup-এ LEGB rule অনুসরণ করে:

1. **Local:** বর্তমান function।
2. **Enclosing:** outer function।
3. **Global:** module level।
4. **Built-in:** Python built-in names।

~~~python
message = "global"

def outer():
    message = "enclosing"

    def inner():
        message = "local"
        print(message)

    inner()

outer()  # local
~~~

~~~text
Local
  |
Enclosing
  |
Global
  |
Built-in
~~~

Python name খুঁজতে এই ক্রমে scope পরীক্ষা করে।

## 23. global এবং nonlocal কী?

**global** দিয়ে function-এর ভেতর থেকে module-level variable reassign করা যায়। **nonlocal** দিয়ে enclosing function-এর variable reassign করা যায়।

~~~python
count = 0

def increment_global():
    global count
    count += 1

increment_global()
print(count)  # 1
~~~

~~~python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1

    inner()
    return count

print(outer())  # 1
~~~

এগুলো assignment-এর scope নির্ধারণ করে। Mutable object mutate করতে সবসময় global বা nonlocal লাগে না।

## 24. Python-এ সবকিছু কি object?

Python-এ integer, string, list, function, class, module-সহ runtime-এর প্রায় সব value object। Function-কে variable-এ রাখা, argument হিসেবে পাঠানো এবং return করা যায়।

~~~python
def greet(name):
    return f"Hello, {name}"

say_hello = greet
print(say_hello("World"))
~~~

Function-কে অন্য function-এর argument হিসেবে পাঠানো যায়। এ কারণেই decorator ও higher-order function সম্ভব।

## 25. Exception কীভাবে handle করবে?

Exception হলো runtime-এ ঘটতে পারে এমন error condition। **try**-তে risky code, **except**-এ নির্দিষ্ট exception handle করা হয়। **else** চলে exception না হলে এবং **finally** cleanup-এর জন্য ব্যবহৃত হয়।

~~~python
try:
    number = int("123")
except ValueError:
    print("Invalid number")
else:
    print("Converted:", number)
finally:
    print("Finished")
~~~

Output:

~~~text
Converted: 123
Finished
~~~

নির্দিষ্ট exception ধরো। অকারণে **except Exception:** দিয়ে সব error চেপে যেও না। নিজের exception তৈরি করা যায়:

~~~python
class InsufficientBalanceError(Exception):
    pass

raise InsufficientBalanceError("Not enough balance")
~~~

## 26. Context Manager কী? with কেন ব্যবহার করি?

Context manager কোনো resource acquire ও release করার lifecycle manage করে। **with** block শেষ হলে cleanup নিশ্চিত করতে সাহায্য করে, এমনকি exception হলেও।

~~~python
with open("notes.txt", "w") as file:
    file.write("Python fundamentals")
~~~

~~~text
Enter context
     |
Resource acquire (file open)
     |
with block executes
     |
Exit context
     |
Resource cleanup (file close)
~~~

নিজের context manager **__enter__** ও **__exit__** method দিয়ে তৈরি করা যায়, অথবা **contextlib** ব্যবহার করা যায়।

## 27. Module, Package এবং __name__ == "__main__" কী?

- **Module:** সাধারণত একটি Python file।
- **Package:** সম্পর্কিত module সংগঠিত করার structure।
- **__name__ == "__main__":** file সরাসরি run হলে condition সাধারণত true হয়; import হলে module-এর নাম থাকে।

~~~python
def main():
    print("Application started")

if __name__ == "__main__":
    main()
~~~

এতে একই file script হিসেবেও চালানো যায়, আবার import করলেও entry-point code অযথা execute হয় না।

## 28. Type hint কী? এটি কি runtime-এ type enforce করে?

Type hint code-এর expected type বোঝায় এবং IDE, static type checker ও developer-কে সাহায্য করে। Python সাধারণত নিজে থেকে runtime-এ type hint enforce করে না।

~~~python
def greet(name: str) -> str:
    return f"Hello, {name}"

print(greet("Shomi"))
print(greet(123))  # Runtime-এ চলতে পারে
~~~

Type checker দিয়ে static analysis করলে type mismatch ধরা সম্ভব।

## 29. Garbage Collection কী?

Garbage collection হলো আর ব্যবহারযোগ্য নয় এমন object-এর memory reclaim করার ব্যবস্থা। CPython প্রধানত reference counting ব্যবহার করে এবং circular reference সামলাতে cyclic garbage collector-ও আছে।

~~~text
Names / containers
       |
       v
     Object
       |
References remain?
   /          \
 Yes           No
  |             |
Keep       Reclaim eligible
~~~

Object-এর reference count শূন্য হলে CPython সাধারণত সেটি reclaim করতে পারে। Circular reference থাকলে cyclic GC প্রয়োজন হতে পারে। সব Python implementation একই memory management strategy ব্যবহার করে না।

## 30. GIL কী?

GIL-এর পূর্ণরূপ Global Interpreter Lock। Traditional GIL-enabled CPython build-এ একই সময়ে একাধিক thread Python bytecode execution-এর নির্দিষ্ট অংশে একসঙ্গে প্রবেশ করতে পারে না।

- **I/O-bound কাজ:** Threading উপকারী হতে পারে, কারণ thread I/O-তে অপেক্ষা করার সময় অন্য thread কাজ করতে পারে।
- **CPU-bound কাজ:** Traditional CPython-এ pure Python CPU work-এর জন্য একাধিক thread সবসময় parallel speedup দেয় না। Multiprocessing বা native code উপযোগী হতে পারে।

Free-threaded CPython build-ও রয়েছে। তাই GIL ব্যাখ্যা করার সময় কোন CPython build বোঝাচ্ছো তা পরিষ্কার করো।

## 31. Threading, Multiprocessing এবং Asyncio-এর পার্থক্য কী?

| Approach | মূল ধারণা | সাধারণ ব্যবহার |
|---|---|---|
| Threading | একই process-এ একাধিক thread | I/O-bound কাজ |
| Multiprocessing | একাধিক process | CPU-bound parallel কাজ |
| Asyncio | event loop ও cooperative concurrency | অনেক concurrent I/O task |

~~~text
Threading       -> এক process, একাধিক thread
Multiprocessing -> একাধিক process
Asyncio         -> event loop, await করে control yield
~~~

কোনটি দ্রুত হবে তা workload, library এবং environment-এর উপর নির্ভর করে। এগুলো interchangeable solution নয়।

## 32. async এবং await কী?

**async def** coroutine function তৈরি করে। **await** কোনো awaitable operation-এর জন্য অপেক্ষার সময় event loop-কে অন্য কাজ চালানোর সুযোগ দেয়।

~~~python
import asyncio

async def fetch_data():
    print("Fetching...")
    await asyncio.sleep(1)
    print("Done")
    return {"status": "ok"}

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())
~~~

await নিজে CPU-bound code-কে parallel করে না। এটি cooperative concurrency, বিশেষ করে I/O অপেক্ষার সময় কার্যকর।

## 33. __str__ এবং __repr__ কী?

**__str__** object-এর user-friendly string representation দেয়। **__repr__** developer/debugging-এর জন্য unambiguous representation দেওয়ার চেষ্টা করে।

~~~python
class User:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

    def __repr__(self):
        return f"User(name={self.name!r})"

user = User("Shomi")
print(str(user))   # Shomi
print(repr(user))  # User(name='Shomi')
~~~

print সাধারণত str ব্যবহার করে। Interactive interpreter-এ object দেখানোর সময় repr গুরুত্বপূর্ণ।

## 34. Class variable এবং Instance variable-এর পার্থক্য কী?

Instance variable প্রতিটি object-এর নিজস্ব state রাখে। Class variable class-এর সঙ্গে যুক্ত এবং instance-গুলোর মধ্যে shared হতে পারে।

~~~python
class User:
    role = "member"  # class variable

    def __init__(self, name):
        self.name = name  # instance variable

first = User("A")
second = User("B")

print(first.name)   # A
print(second.name)  # B
print(first.role)   # member
~~~

Mutable class variable-এর বিপদ:

~~~python
class Team:
    members = []

a = Team()
b = Team()
a.members.append("Shomi")

print(b.members)  # ['Shomi']
~~~

সব instance একই list share করছে। আলাদা list দরকার হলে **__init__**-এ **self.members = []** তৈরি করো।

## 35. Inheritance এবং Composition-এর পার্থক্য কী?

**Inheritance** হলো একটি class অন্য class-এর behavior inherit করে। **Composition** হলো একটি object অন্য object-কে ব্যবহার করে কাজ সম্পন্ন করে।

~~~python
class Animal:
    def speak(self):
        return "sound"

class Dog(Animal):
    pass

print(Dog().speak())
~~~

Composition:

~~~python
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()

    def start(self):
        return self.engine.start()
~~~

“Inheritance হলো is-a, composition হলো has-a” একটি সহায়ক mental model। Reuse-এর জন্য inheritance বাধ্যতামূলক নয়; composition অনেক সময় coupling কমায়।

## 36. staticmethod, classmethod এবং instance method-এর পার্থক্য কী?

| Method | প্রথম parameter | কী access করতে পারে |
|---|---|---|
| Instance method | self | instance ও class |
| Class method | cls | class state |
| Static method | automatic self/cls নেই | নিজে থেকে instance/class state পায় না |

~~~python
class User:
    category = "general"

    def __init__(self, name):
        self.name = name

    def describe(self):
        return self.name

    @classmethod
    def get_category(cls):
        return cls.category

    @staticmethod
    def is_valid_name(name):
        return bool(name.strip())
~~~

Class method alternative constructor বা class-level behavior-এর জন্য ব্যবহৃত হয়। Static method class-এর namespace-এ রাখা related utility function-এর জন্য সুবিধাজনক।

---

# দ্রুত Interview Revision

| প্রশ্ন | এক লাইনের উত্তর |
|---|---|
| Python dynamically typed? | হ্যাঁ, object-এর type runtime-এ থাকে। |
| CPython কী? | Python-এর সবচেয়ে প্রচলিত implementation, মূলত C-তে লেখা। |
| Assignment কি copy? | না, name binding তৈরি করে। |
| Mutable কী? | একই object-এর state পরিবর্তন করা যায়। |
| Immutable কী? | object-এর state তৈরি হওয়ার পর পরিবর্তন করা যায় না। |
| is বনাম ==? | Identity বনাম value equality। |
| Shallow copy? | Outer object নতুন, nested reference share হতে পারে। |
| Deep copy? | Nested structure recursively copy করার চেষ্টা করে। |
| Mutable default argument কেন risky? | Default object function definition-এর সময় একবার তৈরি হয়। |
| *args? | অতিরিক্ত positional arguments-এর tuple। |
| **kwargs? | অতিরিক্ত keyword arguments-এর dict। |
| Generator? | yield দিয়ে একবারে একটি value দেয়। |
| Decorator? | অন্য callable-কে wrap করে behavior extend করে। |
| LEGB? | Local, Enclosing, Global, Built-in। |
| Context manager? | Resource lifecycle ও cleanup manage করে। |
| Hashable? | Stable hash ও equality contract-যুক্ত object। |
| GIL? | Traditional CPython-এ Python execution-এর concurrency সীমিত করে। |
| async/await? | Cooperative asynchronous concurrency-এর syntax। |

# Practice: Interview-এর মতো নিজেকে যাচাই করো

প্রতিটি exercise-এর আগে output predict করবে, তারপর code run করবে এবং কেন এমন output হলো ব্যাখ্যা করবে।

1. দুইটি variable-এ একই list assign করে একটি variable দিয়ে mutation করো। অন্য variable-এ কী দেখা যায়?
2. একই content-এর দুটি list তৈরি করে == এবং is compare করো।
3. Nested dictionary-এর shallow ও deep copy তৈরি করো। Outer key যোগ এবং nested list mutate করে পার্থক্য দেখাও।
4. Mutable default argument-এর bug reproduce করে None pattern দিয়ে fix করো।
5. Tuple-এর মধ্যে list রেখে tuple hash করার চেষ্টা করো। কেন error হয় ব্যাখ্যা করো।
6. Generator তৈরি করে next() দিয়ে একে একে value বের করো। শেষ হলে কী exception হয়?
7. Decorator তৈরি করো, যা function call-এর আগে ও পরে message print করে।
8. LEGB rule বোঝাতে একই name local, enclosing এবং global scope-এ রাখো।
9. Context manager দিয়ে file open করে exception হলেও file বন্ধ হচ্ছে কি না যাচাই করো।
10. Threading, multiprocessing এবং asyncio-এর মধ্যে একটি I/O-bound ও একটি CPU-bound কাজের জন্য কোনটি বেছে নেবে, কারণসহ ব্যাখ্যা করো।

**Interview preparation-এর নিয়ম:** শুধু definition বলবে না। ছোট example দেবে, output predict করবে এবং “কেন?” প্রশ্নের উত্তর দিতে পারবে। এই তিনটি পারলেই concept মুখস্থ নয়, সত্যিই বোঝা হয়েছে।
