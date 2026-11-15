# Decorators and Context Managers

Python-এ অনেক সময় existing function-এর মূল কাজ না বদলে তার চারপাশে logging, timing, authorization বা caching যোগ করতে হয়। আবার file, database connection কিংবা temporary resource ব্যবহার করলে নিশ্চিত করতে হয় কাজ শেষে resource ঠিকভাবে release হয়েছে। Decorator এবং Context Manager এই দুই ধরনের সমস্যার সমাধান করে।

# Part 1: Decorators

## ১. Decorator কী?

Decorator হলো callable, যা অন্য callable-কে input হিসেবে নিয়ে সাধারণত একটি নতুন callable return করে। নতুন callable original function-এর আগে বা পরে অতিরিক্ত behavior চালাতে পারে।

~~~python
from functools import wraps

def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@log_call
def add(a, b):
    return a + b

print(add(2, 3))
~~~

`@log_call` মূলত এই assignment-এর syntactic sugar:

~~~python
def add(a, b):
    return a + b

add = log_call(add)
~~~

~~~mermaid
flowchart TD
    A["Define original function"] --> B["@log_call applied"]
    B --> C["log_call(original)"]
    C --> D["Wrapper function returned"]
    D --> E["add name now refers to wrapper"]
    E --> F["Call add()"]
    F --> G["Wrapper runs extra behavior"]
    G --> H["Original function runs"]
~~~

## ২. Wrapper function-এ *args এবং **kwargs কেন ব্যবহার হয়?

Decorator যেন বিভিন্ন signature-এর function wrap করতে পারে, সে জন্য wrapper সাধারণত `*args` ও `**kwargs` গ্রহণ করে এবং original function-এ forward করে।

~~~python
from functools import wraps

def announce(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Starting")
        return func(*args, **kwargs)
    return wrapper

@announce
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"

print(greet("Mina", greeting="Welcome"))
~~~

Wrapper যদি original function-এর return value ফেরত না দেয়, তাহলে decorated function-এর return behavior বদলে যেতে পারে। Original function exception raise করলে wrapper সেটি handle না করলে exception caller-এর কাছে propagate করবে।

## ৩. functools.wraps কেন প্রয়োজন?

Wrapper function-এর নিজস্ব name, docstring এবং metadata থাকে। `@wraps(func)` original function-এর metadata wrapper-এ copy করে এবং introspection-এর জন্য `__wrapped__` reference যোগ করে।

~~~python
from functools import wraps

def simple_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@simple_decorator
def calculate():
    """Return a calculation result."""
    return 42

print(calculate.__name__)  # calculate
print(calculate.__doc__)   # Return a calculation result.
~~~

**Interview point:** `wraps` original function-এর সব runtime behavior হুবহু restore করে না; এটি metadata preservation ও introspection উন্নত করে।

## ৪. Decorator with arguments কীভাবে কাজ করে?

Decorator-কে configuration দিতে চাইলে আরেকটি outer function লাগে। Outer function configuration নেয়, inner decorator function নেয় original function, আর wrapper call intercept করে।

~~~python
from functools import wraps

def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hi():
    print("Hi")

say_hi()
~~~

~~~mermaid
flowchart TD
    A["@repeat(3)"] --> B["repeat(times=3)"]
    B --> C["decorator function"]
    C --> D["decorator(say_hi)"]
    D --> E["wrapper returned"]
    E --> F["say_hi() invokes wrapper"]
    F --> G["Original function called 3 times"]
~~~

এখানে `repeat(3)` নিজে final decorator নয়; এটি configured decorator তৈরি করে।

## ৫. একাধিক decorator stack করলে order কীভাবে কাজ করে?

একাধিক decorator থাকলে নিচের decorator আগে apply হয়, তারপর উপরেরটি। অর্থাৎ:

~~~python
@first
@second
def work():
    pass
~~~

এটি equivalent:

~~~python
work = first(second(work))
~~~

~~~mermaid
flowchart TD
    A["Original work()"] --> B["second decorator"]
    B --> C["second wrapper"]
    C --> D["first decorator"]
    D --> E["first wrapper"]
    E --> F["Final work name"]
~~~

Order গুরুত্বপূর্ণ, কারণ wrapper composition বদলালে logging, authorization, caching বা transaction behavior-এর order-ও বদলাতে পারে।

## ৬. Decorator-এর বাস্তব ব্যবহার কী?

- Logging ও tracing
- Timing / metrics
- Authentication ও authorization
- Caching
- Retry policy
- Input validation
- Rate limiting

~~~python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(30))
print(fibonacci.cache_info())
~~~

Cache তখনই নিরাপদ যখন function-এর result তার arguments-এর উপর নির্ভরশীল এবং hidden mutable state বা side effect result-কে প্রভাবিত করে না। `lru_cache`-এর arguments hashable হতে হয়।

## ৭. Async function-এর decorator কীভাবে লিখব?

Async function decorate করলে wrapper-টিও async হওয়া দরকার এবং original coroutine-কে `await` করতে হবে।

~~~python
from functools import wraps
import asyncio

def log_async(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        print("Starting async call")
        result = await func(*args, **kwargs)
        print("Finished async call")
        return result
    return wrapper

@log_async
async def fetch_value():
    await asyncio.sleep(0.1)
    return 42

print(asyncio.run(fetch_value()))
~~~

সাধারণ synchronous wrapper coroutine object return করতে পারে, কিন্তু সেটি await না করলে function-এর কাজ execute নাও হতে পারে। Async behavior-এর জন্য wrapper design স্পষ্ট হওয়া জরুরি।

## ৮. Decorator-এর common pitfalls কী?

1. `functools.wraps` বাদ দিলে debugging ও introspection কঠিন হয়।
2. Return value forward না করলে decorated function-এর contract বদলে যায়।
3. Wrapper-এর ভিতরে exception swallow করলে failure লুকিয়ে যেতে পারে।
4. Decorator-এর mutable closure state concurrent call-এ shared হতে পারে।
5. Async function-এর জন্য sync wrapper ব্যবহার করলে coroutine handling ভুল হতে পারে।
6. Parameterized decorator-এর nesting ভুল হলে configuration ও function-এর স্তর গুলিয়ে যায়।

# Part 2: Context Managers

## ৯. Context Manager কী?

Context Manager resource বা execution context-এর setup ও cleanup manage করে। `with` block শুরু হলে resource acquire হয় এবং block শেষ হলে cleanup হয়, exception ঘটলেও।

~~~python
with open("notes.txt", "w", encoding="utf-8") as file:
    file.write("Hello")
~~~

File block থেকে বের হওয়ার সময় close হয়। তাই explicit `file.close()` ভুলে যাওয়ার ঝুঁকি কমে।

~~~mermaid
flowchart TD
    A["Enter with block"] --> B["Acquire resource"]
    B --> C["Execute block"]
    C --> D{"Normal exit or exception?"}
    D --> E["Context manager cleanup"]
    E --> F["Continue or propagate exception"]
~~~

## ১০. Context Manager protocol কী?

Class-based context manager সাধারণত দুটি special method implement করে:

- `__enter__()`: context-এ প্রবেশের সময় চলে এবং `as` variable-এ যে value bind হবে তা return করে।
- `__exit__(exc_type, exc_value, traceback)`: block থেকে বের হওয়ার সময় চলে। Exception না হলে তিনটি argument-ই `None`।

~~~python
class ManagedResource:
    def __enter__(self):
        print("Acquire resource")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Release resource")
        return False

with ManagedResource() as resource:
    print("Use resource")
~~~

`__exit__` truthy value return করলে block-এর exception suppress হয়। False বা None হলে exception propagate করে।

~~~mermaid
sequenceDiagram
    participant W as with statement
    participant M as Context Manager
    W->>M: __enter__()
    M-->>W: resource
    W->>M: execute block
    W->>M: __exit__(type, value, traceback)
    alt __exit__ returns truthy
        W-->>W: suppress exception
    else false / None
        W-->>W: propagate exception
    end
~~~

## ১১. Exception হলেও cleanup কীভাবে নিশ্চিত করব?

Resource release code `__exit__`-এ রাখলে block-এর ভিতরে exception ঘটলেও Python exit method call করে।

~~~python
class ManagedResource:
    def __enter__(self):
        print("Acquire")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Release")
        if exc_type:
            print(f"Exception: {exc_type.__name__}")
        return False

with ManagedResource():
    raise ValueError("Something went wrong")
~~~

এই example-এ cleanup হবে, তারপর ValueError caller-এর কাছে propagate করবে। `__enter__` নিজেই ব্যর্থ হলে `__exit__` call হয় না, কারণ context-এ প্রবেশ সম্পন্ন হয়নি। তাই resource acquisition-এর মাঝখানে partial failure হলে `__enter__`-এর ভিতরেও প্রয়োজনীয় cleanup করতে হবে।

## ১২. contextlib.contextmanager দিয়ে Context Manager কীভাবে বানাব?

`contextlib.contextmanager` decorator দিয়ে generator function-কে context manager বানানো যায়। `yield`-এর আগের অংশ setup, yield-এর পরের অংশ cleanup।

~~~python
from contextlib import contextmanager

@contextmanager
def managed_connection():
    print("Open connection")
    try:
        yield "connection"
    finally:
        print("Close connection")

with managed_connection() as connection:
    print("Using", connection)
~~~

Block-এর exception yield point-এ throw করা হয়। তাই cleanup-এর জন্য `try/finally` ব্যবহার করা গুরুত্বপূর্ণ।

~~~mermaid
flowchart TD
    A["Call contextmanager function"] --> B["Setup code"]
    B --> C["yield resource"]
    C --> D["with block executes"]
    D --> E["Resume generator"]
    E --> F["finally / cleanup"]
~~~

## ১৩. একাধিক Context Manager কীভাবে ব্যবহার করব?

একাধিক resource manage করতে একাধিক context manager ব্যবহার করা যায়।

~~~python
with open("input.txt", encoding="utf-8") as source, \\
     open("output.txt", "w", encoding="utf-8") as target:
    target.write(source.read())
~~~

এগুলো nested with block-এর মতো কাজ করে। Exit reverse order-এ হয়: সর্বশেষে acquire করা resource আগে release হয়।

~~~mermaid
flowchart TD
    A["Acquire input file"] --> B["Acquire output file"]
    B --> C["Execute block"]
    C --> D["Close output file"]
    D --> E["Close input file"]
~~~

## ১৪. ExitStack কখন দরকার?

যখন runtime-এ কতগুলো resource acquire হবে তা আগে জানা নেই, তখন `contextlib.ExitStack` উপকারী। এটি context manager-গুলো stack করে এবং exit-এর সময় reverse order-এ cleanup করে।

~~~python
from contextlib import ExitStack

paths = ["a.txt", "b.txt", "c.txt"]

with ExitStack() as stack:
    files = [
        stack.enter_context(open(path, encoding="utf-8"))
        for path in paths
    ]
    for file in files:
        print(file.readline())
~~~

বাস্তব code-এ file-not-found এবং partial acquisition failure-ও handle করতে হবে। ExitStack আগে সফলভাবে acquire হওয়া resource-গুলো release করতে সাহায্য করে।

## ১৫. Async Context Manager কী?

Async resource lifecycle manage করতে `async with` ব্যবহার হয়। Async context manager-এ `__aenter__()` ও `__aexit__()` awaitable return করে।

~~~python
class AsyncResource:
    async def __aenter__(self):
        print("Async acquire")
        return self

    async def __aexit__(self, exc_type, exc_value, traceback):
        print("Async cleanup")
        return False

async def main():
    async with AsyncResource():
        print("Using async resource")
~~~

~~~mermaid
sequenceDiagram
    participant C as Async caller
    participant R as Async resource
    C->>R: await __aenter__()
    R-->>C: resource
    C->>C: await block operations
    C->>R: await __aexit__(...)
    R-->>C: cleanup complete
~~~

HTTP client session, async database session এবং অন্যান্য asynchronous resource-এর lifecycle manage করতে এটি কাজে লাগে।

## ১৬. Decorator ও Context Manager-এর মধ্যে পার্থক্য কী?

| Decorator | Context Manager |
|---|---|
| Callable-কে wrap করে behavior যোগ করে | নির্দিষ্ট block-এর lifecycle manage করে |
| Function/class definition-এ বা callable-এর উপর apply হয় | `with` / `async with` block-এ ব্যবহৃত |
| Logging, caching, authorization | File, connection, lock, transaction |
| Callable return/replace করতে পারে | Enter value ও exit cleanup control করে |

দুটোই cross-cutting concern manage করতে পারে, কিন্তু ব্যবহারের boundary আলাদা।

## AI Engineering ও Backend Development-এ ব্যবহার

- Decorator দিয়ে API endpoint authorization, tracing ও metrics।
- Context manager দিয়ে database session/transaction lifecycle।
- Context manager দিয়ে file ও temporary resource cleanup।
- Async context manager দিয়ে HTTP client বা async database session।
- Retry decorator দিয়ে transient failure retry, তবে idempotency ও retry limit বিবেচনা করতে হবে।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Decorator কী? | Callable wrap করে behavior যোগ করে |
| `@decorator` equivalent কী? | `func = decorator(func)` |
| `wraps` কেন? | Original function metadata preserve করতে |
| Decorator arguments কীভাবে নেয়? | Outer function configuration নেয়, inner decorator function wrap করে |
| Multiple decorator order? | নিচের decorator আগে apply হয় |
| Async decorator-এ কী দরকার? | সাধারণত async wrapper ও await |
| Context manager কী? | Resource/context setup ও cleanup manage করে |
| Protocol methods? | `__enter__`, `__exit__` |
| `__exit__` কখন exception suppress করে? | Truthy return করলে |
| `contextmanager` কী করে? | Generator-কে context manager বানায় |
| `ExitStack` কেন? | Dynamic সংখ্যক context/resource manage করতে |
| Async context manager methods? | `__aenter__`, `__aexit__` |

## Practice Exercises

1. Function execution time মাপে এমন decorator লেখো। Return value ও exception behavior যেন বদলে না যায়।
2. একটি decorator লেখো যা function call count করে। Counter কোথায় রাখা হচ্ছে এবং concurrent call-এ কী হতে পারে তা ব্যাখ্যা করো।
3. Parameterized decorator লেখো যা function সর্বোচ্চ N বার retry করে। কোন exception retry হবে তা configurable করো।
4. একাধিক decorator stack করো এবং print দিয়ে execution order observe করো।
5. Async function-এর জন্য timing decorator লেখো।
6. Class-based context manager লেখো যা resource acquire/release message print করে।
7. Context manager-এর ভিতরে exception raise করে পরীক্ষা করো cleanup হয় কি না এবং exception propagate করে কি না।
8. `contextlib.contextmanager` দিয়ে temporary resource manage করো।
9. `ExitStack` ব্যবহার করে runtime-এ পাওয়া file path-গুলোর file open করো।
10. Async context manager implement করো এবং `async with` দিয়ে ব্যবহার করো।

## Practice করার পদ্ধতি

1. Decorator-এর ক্ষেত্রে original function-এর input, output, metadata এবং exception behavior পরীক্ষা করো।
2. Context manager-এর ক্ষেত্রে normal exit, exceptional exit এবং acquisition failure পরীক্ষা করো।
3. Resource cleanup ঠিক একবার হচ্ছে কি না যাচাই করো।
4. কোন behavior decorator দিয়ে এবং কোনটি context manager দিয়ে model করা ভালো, তা ব্যাখ্যা করো।
