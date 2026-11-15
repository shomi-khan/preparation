# Iterators and Generators

Python-এ data process করার একটি গুরুত্বপূর্ণ প্রশ্ন হলো: **সব data একবারে memory-তে আনব, নাকি প্রয়োজন অনুযায়ী একটির পর একটি নেব?** Iterator ও Generator এই দ্বিতীয় পদ্ধতিটিকে সম্ভব করে। বড় file, database result, event stream কিংবা infinite sequence process করার সময় এগুলো বিশেষভাবে কাজে লাগে।

## ১. Iterable কী?

যে object থেকে `iter()` ব্যবহার করে iterator পাওয়া যায়, সেটি iterable। যেমন: list, tuple, string, dictionary, set এবং file object।

~~~python
names = ["Asha", "Rafi", "Mina"]

iterator = iter(names)
print(next(iterator))  # Asha
~~~

Iterable নিজে data-এর collection বা source হতে পারে। এটি সাধারণত বলে দেয় কীভাবে তার item-গুলো iterate করতে হবে।

~~~mermaid
flowchart TD
    A["Iterable: list / tuple / file"] --> B["iter(iterable)"]
    B --> C["Iterator object"]
    C --> D["next(iterator)"]
    D --> E["Next item"]
~~~

## ২. Iterator কী?

Iterator এমন object যা একবারে একটি item দেয়। Python iterator protocol অনুযায়ী এর দুটি method থাকে:

- `__iter__()`: iterator object-টিই return করে।
- `__next__()`: পরের item return করে। item শেষ হলে `StopIteration` raise করে।

~~~python
numbers = [10, 20, 30]
iterator = iter(numbers)

print(next(iterator))  # 10
print(next(iterator))  # 20
print(next(iterator))  # 30

# next(iterator)  # StopIteration
~~~

Iterator নিজের progress বা state ধরে রাখে। তাই পরের `next()` আগের item নয়, পরবর্তী item দেয়।

~~~mermaid
sequenceDiagram
    participant C as Caller
    participant I as Iterator
    C->>I: next()
    I-->>C: 10
    C->>I: next()
    I-->>C: 20
    C->>I: next()
    I-->>C: 30
    C->>I: next()
    I-->>C: StopIteration
~~~

## ৩. Iterable এবং Iterator-এর পার্থক্য কী?

| Iterable | Iterator |
|---|---|
| Iterator তৈরি করার source | একবারে একটি item দেয় |
| `iter(obj)` করা যায় | `next(obj)` করা যায় |
| List সাধারণ iterable | `iter(list)` থেকে পাওয়া object iterator |
| অনেক ক্ষেত্রে বারবার iterate করা যায় | সাধারণত একবার consume হয় |

একটি list থেকে একাধিক independent iterator তৈরি করা যায়:

~~~python
numbers = [10, 20, 30]
first = iter(numbers)
second = iter(numbers)

print(next(first))   # 10
print(next(first))   # 20
print(next(second))  # 10, independent position
~~~

একটি iterator নিজেও iterable। সাধারণত iterator-এর `__iter__()` নিজের reference return করে।

~~~mermaid
flowchart LR
    L["List: [10, 20, 30]"] --> I1["Iterator A<br/>position 2"]
    L --> I2["Iterator B<br/>position 1"]
    I1 --> N1["next() → 20"]
    I2 --> N2["next() → 10"]
~~~

## ৪. for loop কীভাবে iterator ব্যবহার করে?

`for` loop internally iterable থেকে iterator নেয়, তারপর বারবার `next()` call করে। `StopIteration` পেলে loop শেষ হয়।

~~~python
for name in ["Asha", "Rafi", "Mina"]:
    print(name)
~~~

Conceptually এটি এমন:

~~~python
iterator = iter(["Asha", "Rafi", "Mina"])

while True:
    try:
        name = next(iterator)
    except StopIteration:
        break
    print(name)
~~~

~~~mermaid
flowchart TD
    A["for item in iterable"] --> B["iterator = iter(iterable)"]
    B --> C["next(iterator)"]
    C --> D{"StopIteration?"}
    D -->|না| E["Loop body executes"]
    E --> C
    D -->|হ্যাঁ| F["Loop ends"]
~~~

## ৫. Generator কী?

Generator হলো iterator তৈরির সহজ উপায়। Generator function-এ `yield` ব্যবহার করা হয়। Function call করলে function body সঙ্গে সঙ্গে execute হয় না; একটি generator object return হয়।

~~~python
def count_up_to(limit):
    number = 1
    while number <= limit:
        yield number
        number += 1

counter = count_up_to(3)
print(next(counter))  # 1
print(next(counter))  # 2
print(next(counter))  # 3
~~~

প্রতিটি `yield`-এ function execution pause হয়। পরের `next()` call-এ আগের অবস্থান থেকে execution resume হয়।

~~~mermaid
flowchart TD
    A["count_up_to(3)"] --> B["Generator object তৈরি"]
    B --> C["next()"]
    C --> D["yield 1, pause"]
    D --> E["next()"]
    E --> F["yield 2, pause"]
    F --> G["next()"]
    G --> H["yield 3, pause"]
    H --> I["next()"]
    I --> J["Function শেষ: StopIteration"]
~~~

## ৬. yield এবং return-এর পার্থক্য কী?

- `return` function execution শেষ করে এবং একটি result ফেরত দেয়।
- `yield` value দেয়, কিন্তু function-এর execution state ধরে রেখে pause করে।
- Generator function-এ `return value` generator iteration শেষ করে। সেই value `StopIteration.value`-এর মাধ্যমে পাওয়া যায়, সাধারণ `for` loop সেটি consume করে না।

~~~python
def example():
    yield "first"
    yield "second"
    return "finished"

g = example()
print(next(g))  # first
print(next(g))  # second

try:
    next(g)
except StopIteration as error:
    print(error.value)  # finished
~~~

## ৭. Generator কেন ব্যবহার করব?

Generator lazy evaluation করে: প্রয়োজন না হওয়া পর্যন্ত পরের value তৈরি করে না। বড় file বা stream sequentially process করতে এটি memory সাশ্রয় করতে পারে।

~~~python
def read_numbers(path):
    with open(path, encoding="utf-8") as file:
        for line in file:
            line = line.strip()
            if line:
                yield int(line)

for number in read_numbers("numbers.txt"):
    print(number)
~~~

এখানে পুরো file একসঙ্গে list-এ load করা হচ্ছে না। প্রতিবার একটি line read ও process করা হচ্ছে।

~~~mermaid
flowchart LR
    A["Large file"] --> B["Read one line"]
    B --> C["Convert / process"]
    C --> D["Yield result"]
    D --> E["Consumer"]
    E -. "next value needed" .-> B
~~~

**সতর্কতা:** Generator নিজে সব সমস্যার সমাধান নয়। Consumer যদি সব value list-এ জমা করে, তাহলে memory saving থাকবে না।

## ৮. List comprehension বনাম Generator expression

~~~python
squares_list = [n * n for n in range(1_000_000)]
squares_gen = (n * n for n in range(1_000_000))
~~~

| List comprehension | Generator expression |
|---|---|
| সব result সঙ্গে সঙ্গে তৈরি করে | প্রয়োজন অনুযায়ী result তৈরি করে |
| পুরো result memory-তে রাখে | সাধারণত অল্প state রাখে |
| বারবার iterate করা যায় | একবার consume হয় |
| indexing করা যায় | সরাসরি indexing করা যায় না |

~~~mermaid
flowchart TD
    A["Comprehension"] --> B["List comprehension"]
    A --> C["Generator expression"]
    B --> D["সব value তৈরি<br/>List in memory"]
    C --> E["Generator object"]
    E --> F["next()"]
    F --> G["একটি value তৈরি"]
    G --> H{"আর value দরকার?"}
    H -->|হ্যাঁ| F
    H -->|না| I["Stop"]
~~~

Generator expression ব্যবহার করো যখন একবার sequentially process করাই লক্ষ্য। Result বারবার ব্যবহার, index বা random access দরকার হলে list ভালো।

## ৯. Generator কি সবসময় দ্রুত বা memory-efficient?

না। Generator memory-তে পুরো result ধরে রাখে না, কিন্তু প্রতিটি item produce করার overhead থাকে। ছোট dataset-এ list দ্রুতও হতে পারে। Performance workload অনুযায়ী measure করতে হবে।

আরেকটি বিষয়: `sys.getsizeof(generator)` generator-এর retained object size দেখায়, ভবিষ্যতে produce হবে এমন সব value-এর hypothetical memory নয়। List-এর size-ও nested objects-এর সম্পূর্ণ memory অন্তর্ভুক্ত করে না।

## ১০. Generator exhausted হলে কী হয়?

একবার generator শেষ হয়ে গেলে একই object থেকে আর value পাওয়া যায় না।

~~~python
def values():
    yield 1
    yield 2

g = values()
print(list(g))  # [1, 2]
print(list(g))  # []
~~~

আবার শুরু থেকে দরকার হলে generator function আবার call করতে হবে:

~~~python
first_run = values()
second_run = values()
~~~

## ১১. yield from কী?

`yield from` একটি iterable বা generator-এর item-গুলো একে একে yield করার সংক্ষিপ্ত উপায়।

~~~python
def combined():
    yield from [1, 2]
    yield from (3, 4)

print(list(combined()))  # [1, 2, 3, 4]
~~~

Nested generator delegate করার সময় এটি শুধু loop-এর syntax কমায় না; subgenerator-এর return value-ও গ্রহণ করতে পারে।

~~~python
def child():
    yield "child item"
    return "child result"

def parent():
    result = yield from child()
    yield result

print(list(parent()))
# ['child item', 'child result']
~~~

~~~mermaid
flowchart TD
    A["Parent generator"] --> B["yield from child()"]
    B --> C["Child yields items"]
    C --> D["Parent forwards each item"]
    C --> E["Child returns final value"]
    E --> F["Parent receives return value"]
~~~

## ১২. Generator-এর send(), throw() এবং close() কী?

Generator শুধু value বাইরে পাঠায় না; consumer generator-এর সঙ্গে কিছুটা communication-ও করতে পারে।

- `send(value)`: suspended generator-এর `yield` expression-এ value পাঠায়।
- `throw(exception)`: generator-এর suspended অবস্থানে exception raise করে।
- `close()`: generator-এ `GeneratorExit` পাঠিয়ে বন্ধ করে; `finally` block cleanup করতে পারে।

~~~python
def echo():
    received = yield "ready"
    yield received

g = echo()
print(next(g))       # ready
print(g.send("hello"))  # hello
~~~

~~~python
def managed():
    try:
        yield "resource in use"
    finally:
        print("cleanup")

g = managed()
print(next(g))
g.close()  # cleanup
~~~

এগুলো advanced generator control। সাধারণ data pipeline-এ শুধু `yield` ও iteration-ই বেশি ব্যবহৃত হয়।

## ১৩. Infinite generator কী?

যে generator-এর natural end নেই, সেটি infinite generator। এটি প্রয়োজন অনুযায়ী value produce করে; consumer-কে নিজে ঠিক করতে হয় কখন থামবে।

~~~python
def integers():
    number = 0
    while True:
        yield number
        number += 1

for number in integers():
    if number == 5:
        break
    print(number)
~~~

Infinite generator-কে সরাসরি `list()` দিয়ে consume করলে program থামবে না এবং memory শেষ হতে পারে।

## ১৪. itertools কেন গুরুত্বপূর্ণ?

`itertools` standard library-তে efficient iterator building blocks দেয়।

~~~python
from itertools import islice, count, chain

first_five = list(islice(count(10), 5))
combined = list(chain([1, 2], [3, 4]))

print(first_five)  # [10, 11, 12, 13, 14]
print(combined)    # [1, 2, 3, 4]
~~~

Interview-এ `islice`, `chain`, `count`, `cycle`, `groupby`-এর use case জানা উপকারী। `groupby` consecutive matching keys group করে; global grouping চাইলে আগে sort করতে হতে পারে।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Iterable কী? | যেখান থেকে iterator পাওয়া যায় |
| Iterator কী? | `__iter__` ও `__next__` protocol মেনে চলে |
| Iterator শেষ হলে? | `StopIteration` |
| for loop কী করে? | Iterator থেকে বারবার next নেয় |
| Generator কী? | `yield`-সহ function থেকে তৈরি iterator |
| Generator function call করলে কী হয়? | Generator object return হয়; body তখনই চলে না |
| yield বনাম return? | yield pause করে; return function শেষ করে |
| Generator reusable? | একই exhausted object পুনরায় শুরু হয় না |
| yield from? | অন্য iterable/generator-এর item delegate করে |
| Infinite generator-এর ঝুঁকি? | সীমাহীন consume করলে loop/memory issue |
| Generator সবসময় দ্রুত? | না; memory ও runtime trade-off আছে |

## Practice Exercises

1. ১ থেকে N পর্যন্ত even number yield করে এমন generator লেখো। N negative, zero ও positive হলে behavior define করো।
2. একটি file line-by-line read করে blank line বাদ দিয়ে non-empty line yield করো।
3. একটি generator লেখো যা বড় CSV file-এর row একে একে yield করে। Header handling এবং invalid row policy define করো।
4. একাধিক iterable combine করতে `yield from` ব্যবহার করো।
5. Infinite integer generator থেকে `itertools.islice` দিয়ে প্রথম ২০টি value নাও।
6. `send()` ব্যবহার করে একটি running total generator implement করো।
7. Generator-এর `finally` block কখন চলে তা `close()` ও normal exhaustion দিয়ে observe করো।
8. একই data list ও generator দিয়ে process করে memory ও runtime তুলনা করো। Measurement limitation ব্যাখ্যা করো।
9. দুইটি generator pipeline chain করে intermediate list ছাড়াই sum বের করো।
10. `itertools.groupby` ব্যবহার করে consecutive repeated item group করো। তারপর unsorted input-এ behavior পরীক্ষা করো।

## Practice করার পদ্ধতি

1. Code লেখার আগে প্রতিটি `next()` call-এর output predict করো।
2. Empty iterable, single item, exhausted iterator এবং infinite generator test করো।
3. Generator কখন execute হয় এবং কখন pause হয় তা নিজের ভাষায় explain করো।
4. Memory saving-এর পাশাপাশি execution overhead ও consumption behavior বিবেচনা করো।
5. Notebook-এ observation এবং conclusion লিখে রাখো।
