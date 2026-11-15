# Exception Handling and Type Hints

Production code শুধু successful input-এর জন্য লেখা হয় না। Invalid input, network failure, missing file, timeout কিংবা unexpected state-এর সময় code কীভাবে behave করবে, সেটিও design-এর অংশ। Exception handling failure path সামলায়; type hints code-এর contract আরও পরিষ্কার করে এবং static analysis-কে সাহায্য করে।

# Part 1: Exception Handling

## ১. Exception কী?

Program চলার সময় কোনো operation স্বাভাবিকভাবে complete হতে না পারলে Python exception raise করতে পারে। Exception handle না হলে current call stack দিয়ে উপরে propagate হয় এবং শেষ পর্যন্ত uncaught থাকলে program/thread-এর execution ব্যর্থ হয়।

~~~python
try:
    value = int("not-a-number")
except ValueError as error:
    print(f"Invalid integer: {error}")
~~~

~~~mermaid
flowchart TD
    A["Execute operation"] --> B{"Exception raised?"}
    B -->|না| C["Continue normally"]
    B -->|হ্যাঁ| D["Search matching except handler"]
    D --> E{"Handler found?"}
    E -->|হ্যাঁ| F["Handle / recover"]
    E -->|না| G["Propagate to caller"]
    G --> H["Uncaught: execution fails"]
~~~

## ২. try, except, else এবং finally কীভাবে কাজ করে?

- `try`: যে code exception raise করতে পারে।
- `except`: matching exception handle করে।
- `else`: try block-এ exception না হলে execute হয়।
- `finally`: সাধারণত success বা failure উভয় ক্ষেত্রেই cleanup-এর জন্য execute হয়।

~~~python
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Cannot divide by zero")
else:
    print("Result:", result)
finally:
    print("Operation finished")
~~~

~~~mermaid
flowchart TD
    A["try block"] --> B{"Exception?"}
    B -->|হ্যাঁ| C["Matching except"]
    B -->|না| D["else block"]
    C --> E["finally block"]
    D --> E
    E --> F["Continue / propagate"]
~~~

**সতর্কতা:** `finally` cleanup-এর জন্য। `finally`-এর ভিতরে `return` দিলে pending exception বা আগের return override হতে পারে, তাই সাধারণত এড়িয়ে চলো। Process forcibly terminate হলে বা interpreter shutdown-এর কিছু পরিস্থিতিতে cleanup guarantee-ও সীমিত।

## ৩. নির্দিষ্ট Exception ধরবে কেন?

যে failure তুমি recover করতে পারবে, সেটিই handle করো। নির্বিচারে `except Exception:` বা `except:` ব্যবহার করলে programming bug লুকিয়ে যেতে পারে।

খারাপ:

~~~python
try:
    process_data()
except Exception:
    pass
~~~

ভালো:

~~~python
try:
    user_id = int(raw_user_id)
except ValueError as error:
    raise ValueError("user_id must be an integer") from error
~~~

একাধিক expected exception একইভাবে handle করা যায়:

~~~python
try:
    value = int(raw_value)
except (TypeError, ValueError) as error:
    print(f"Invalid input: {error}")
~~~

## ৪. Exception কীভাবে propagate হয়?

একটি function exception handle না করলে caller-এর কাছে propagate হয়। Call stack-এর উপরের কোনো function matching exception handle করলে সেখানে recovery হতে পারে।

~~~python
def load_config():
    return int("invalid")

def start_app():
    return load_config()

try:
    start_app()
except ValueError:
    print("Configuration is invalid")
~~~

~~~mermaid
flowchart TD
    A["start_app()"] --> B["load_config()"]
    B --> C["int('invalid') raises ValueError"]
    C --> D["load_config has no handler"]
    D --> E["Exception propagates to start_app caller"]
    E --> F["Caller catches ValueError"]
    F --> G["Recovery / useful error response"]
~~~

## ৫. raise এবং raise from কী?

`raise` দিয়ে exception তৈরি বা re-raise করা যায়।

~~~python
def validate_age(age):
    if age < 0:
        raise ValueError("age cannot be negative")
~~~

Exception-এর ভিতর থেকে bare `raise` করলে current exception-এর traceback বজায় রেখে re-raise হয়।

~~~python
try:
    int("abc")
except ValueError:
    print("Could not parse")
    raise
~~~

Low-level exception-কে domain-specific exception-এ রূপান্তর করলে `raise ... from ...` দিয়ে original cause preserve করা যায়।

~~~python
class ConfigurationError(Exception):
    pass

try:
    port = int("not-a-port")
except ValueError as error:
    raise ConfigurationError("Invalid PORT configuration") from error
~~~

~~~mermaid
flowchart TD
    A["Low-level operation"] --> B["ValueError"]
    B --> C["Catch original error"]
    C --> D["Raise ConfigurationError from ValueError"]
    D --> E["Traceback shows high-level error and cause"]
~~~

## ৬. Custom Exception কখন তৈরি করব?

Application বা domain-এর নিজস্ব failure condition স্পষ্টভাবে প্রকাশ করতে custom exception তৈরি করো। সাধারণত `Exception` বা domain-specific base exception inherit করাই যথেষ্ট।

~~~python
class AppError(Exception):
    """Base exception for application errors."""

class UserNotFoundError(AppError):
    pass

def get_user(user_id: int):
    if user_id != 1:
        raise UserNotFoundError(f"User {user_id} was not found")
    return {"id": 1, "name": "Mina"}
~~~

Caller চাইলে specific exception বা পুরো domain family catch করতে পারে:

~~~python
try:
    get_user(99)
except UserNotFoundError:
    print("Show not-found response")
except AppError:
    print("Handle another application error")
~~~

## ৭. Exception hierarchy কী?

Python-এর built-in exception-গুলোর inheritance hierarchy আছে। বেশিরভাগ application error-এর জন্য `Exception` base class। `KeyboardInterrupt`, `SystemExit` এবং `GeneratorExit` হলো `BaseException`-এর direct subclass, তাই সাধারণ `except Exception` এগুলো ধরে না।

~~~mermaid
flowchart TD
    A["BaseException"] --> B["Exception"]
    A --> C["KeyboardInterrupt"]
    A --> D["SystemExit"]
    A --> E["GeneratorExit"]
    B --> F["ValueError"]
    B --> G["TypeError"]
    B --> H["OSError"]
    B --> I["RuntimeError"]
~~~

সাধারণ application code-এ `except BaseException` এড়িয়ে চলো; এতে user interrupt ও process exit-এর মতো control-flow signal-ও ধরে ফেলতে পারো।

## ৮. ExceptionGroup এবং except* কী?

Python 3.11+ এ একসঙ্গে একাধিক exception represent করতে `ExceptionGroup` ব্যবহার করা যায়। `except*` group-এর matching exception-গুলো handle করে।

~~~python
errors = ExceptionGroup(
    "validation errors",
    [ValueError("bad age"), TypeError("bad name")]
)

try:
    raise errors
except* ValueError as group:
    print("Value errors:", group.exceptions)
except* TypeError as group:
    print("Type errors:", group.exceptions)
~~~

এটি concurrent task বা একাধিক independent operation থেকে একাধিক failure একসঙ্গে ফেরত আসার ক্ষেত্রে উপকারী। Basic interview-এর জন্য ধারণাটি জানা যথেষ্ট; বাস্তবে ব্যবহার করার আগে ExceptionGroup-এর propagation semantics বুঝে নাও।

# Part 2: Type Hints

## ৯. Type Hints কী?

Type hints function parameter, return value এবং variable-এর expected type প্রকাশ করে। এগুলো readability, IDE autocomplete এবং static analysis উন্নত করে। Python runtime সাধারণভাবে annotation enforce করে না।

~~~python
def calculate_total(prices: list[float]) -> float:
    return sum(prices)

total: float = calculate_total([10.5, 20.0])
~~~

Static checker যেমন mypy বা pyright সম্ভাব্য type mismatch development time-এ ধরতে পারে।

~~~mermaid
flowchart TD
    A["Python source + type hints"] --> B["Static type checker"]
    B --> C["Potential type errors reported"]
    A --> D["Python runtime"]
    D --> E["Annotations are not automatically enforced"]
    E --> F["Explicit validation needed for untrusted input"]
~~~

## ১০. Union এবং Optional type কী?

একটি value একাধিক type-এর হতে পারে বোঝাতে union ব্যবহার হয়। Python 3.10+ এ `|` syntax ব্যবহার করা যায়।

~~~python
def normalize(value: str | int) -> str:
    return str(value)

def find_user(user_id: int) -> str | None:
    if user_id == 1:
        return "Mina"
    return None
~~~

`Optional[str]` মানে `str | None`, অর্থাৎ value string অথবা None হতে পারে। Optional মানে argument বাদ দেওয়া যাবে এমন নয়; default value না থাকলে argument এখনও required।

## ১১. Any এবং object-এর পার্থক্য কী?

- `Any` type checker-কে বলে value-এর type সম্পর্কে check সীমিত করো; যেকোনো operation অনুমোদিত হতে পারে।
- `object` হলো Python-এর সব object-এর common base type; এর উপর শুধু object-এর জন্য নিরাপদ operation করা যায়, অন্য operation-এর আগে type narrow করতে হয়।

~~~python
from typing import Any

def loose(value: Any) -> None:
    value.not_a_real_method()  # static checker সাধারণত বাধা দেবে না

def safe(value: object) -> None:
    # value.upper()  # checker আপত্তি করবে
    if isinstance(value, str):
        print(value.upper())
~~~

Type safety বজায় রাখতে অপ্রয়োজনে `Any` ছড়িয়ে না দিয়ে `object` ও explicit type narrowing বিবেচনা করো।

## ১২. Built-in generic type hints কী?

Modern Python-এ collection type hint-এ built-in generic syntax ব্যবহার করা যায়:

~~~python
def names_by_id(users: dict[int, str]) -> list[str]:
    return list(users.values())

def unique_tags(tags: set[str]) -> set[str]:
    return tags
~~~

Python 3.9+ এ `list[str]`, `dict[str, int]` ইত্যাদি ব্যবহার করা যায়। পুরোনো version-এ `typing.List`, `typing.Dict` দেখা যেতে পারে।

## ১৩. TypedDict কী?

Dictionary-shaped data-এর key ও value type describe করতে `TypedDict` ব্যবহার হয়। এটি static typing-এর জন্য; runtime-এ dictionary validate করে না।

~~~python
from typing import TypedDict, NotRequired

class UserPayload(TypedDict):
    id: int
    name: str
    nickname: NotRequired[str]

payload: UserPayload = {"id": 7, "name": "Mina"}
~~~

`NotRequired` Python 3.11+ এ `typing` থেকে পাওয়া যায়। আগের version-এ `typing_extensions` লাগতে পারে।

## ১৪. Dataclass কী এবং TypedDict থেকে পার্থক্য কী?

`dataclass` data-holding class তৈরির boilerplate কমায়; যেমন `__init__`, `__repr__`, equality method ইত্যাদি generate করতে পারে।

~~~python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str

user = User(id=7, name="Mina")
print(user)
~~~

| TypedDict | Dataclass |
|---|---|
| Dictionary-এর shape type করে | Class instance তৈরি করে |
| Runtime-এ plain dict | Runtime-এ class instance |
| Static checker-এর সহায়তা | Generated methods ও data container behavior |
| নিজে runtime validation করে না | Type annotation নিজে enforce করে না |

## ১৫. Protocol কী?

Protocol structural typing দেয়: কোনো class নির্দিষ্ট method/attribute implement করলে explicit inheritance ছাড়াও সেই protocol satisfy করতে পারে।

~~~python
from typing import Protocol

class SupportsClose(Protocol):
    def close(self) -> None: ...

def release(resource: SupportsClose) -> None:
    resource.close()
~~~

এটি duck typing-এর static typing counterpart হিসেবে ভাবতে পারো। Type checker structure দেখে compatibility যাচাই করে।

## ১৬. TypeVar ও Generic কী?

একটি function বা class একাধিক type-এর জন্য reusable রাখতে Generic ব্যবহার হয়। TypeVar input ও output type-এর সম্পর্ক preserve করতে সাহায্য করে।

~~~python
from typing import TypeVar

T = TypeVar("T")

def first_item(items: list[T]) -> T:
    return items[0]

name = first_item(["Mina", "Rafi"])  # inferred str
number = first_item([10, 20])        # inferred int
~~~

Generic class-ও তৈরি করা যায়, যেমন typed repository বা container। Interview-এ মূল ধারণা হলো: Generic একই implementation বিভিন্ন type-এ ব্যবহার করেও type relationship ধরে রাখে।

## ১৭. Callable, Literal এবং overload কী কাজে লাগে?

- `Callable`: function বা callable object-এর expected signature বোঝায়।
- `Literal`: নির্দিষ্ট value-গুলোর একটি বোঝায়।
- `@overload`: একই function-এর বিভিন্ন input signature ও corresponding return type static checker-কে বোঝায়।

~~~python
from collections.abc import Callable
from typing import Literal

def apply(value: int, operation: Callable[[int], int]) -> int:
    return operation(value)

Mode = Literal["read", "write"]
~~~

`overload` advanced typing-এ useful যখন input type অনুযায়ী return type বদলায়। Runtime implementation সাধারণত একটিই থাকে।

## ১৮. Type hints কি runtime validation?

না। নিচের function ভুল type দিয়েও call করা যায়; Python নিজে annotation enforce করে না।

~~~python
def greet(name: str) -> str:
    return "Hello " + name

# greet(123)  # Runtime-এ TypeError হতে পারে, type hint নিজে আগে আটকায় না
~~~

External input যেমন HTTP request, JSON, environment variable বা CSV row validate করতে explicit parsing/validation দরকার। Pydantic-এর মতো library runtime validation করতে পারে।

~~~mermaid
flowchart TD
    A["Untrusted JSON / API input"] --> B["Parse and validate"]
    B --> C{"Valid?"}
    C -->|হ্যাঁ| D["Typed application data"]
    C -->|না| E["Validation error"]
    D --> F["Business logic"]
~~~

**মনে রাখবে:** Type hints, static type checker এবং runtime validator তিনটি আলাদা জিনিস।

## ১৯. Type narrowing কী?

কোনো condition বা runtime check দিয়ে broad type-কে নির্দিষ্ট type-এ সীমাবদ্ধ করাকে type narrowing বলে।

~~~python
def display(value: str | int) -> str:
    if isinstance(value, str):
        return value.upper()
    return str(value)
~~~

`isinstance`, `is None`, pattern matching এবং কিছু typing helper checker-কে type narrow করতে সাহায্য করে।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Exception কী? | Runtime exceptional condition signal |
| try/except? | Exception handle করার mechanism |
| else কখন চলে? | try-তে exception না হলে |
| finally কেন? | Cleanup |
| bare raise? | Current exception re-raise |
| raise from? | Exception cause chain preserve |
| Custom exception? | Domain-specific failure represent করে |
| BaseException ধরব? | সাধারণত না |
| Type hints runtime enforce করে? | না |
| Optional[str]? | str অথবা None |
| Any বনাম object? | Any type checks শিথিল করে; object-এ narrowing দরকার |
| TypedDict? | Dictionary shape-এর static typing |
| Dataclass? | Data class boilerplate কমায় |
| Protocol? | Structural typing contract |
| Generic? | বিভিন্ন type-এর জন্য reusable typed code |
| Runtime validation? | Explicit validator/library দিয়ে |

## Practice Exercises

1. String থেকে integer parse করো। Empty string, whitespace, invalid input এবং negative value-এর policy নির্ধারণ করো।
2. Domain-specific `InvalidAmountError` তৈরি করে negative amount reject করো।
3. Low-level `ValueError`-কে custom exception-এ `raise ... from ...` দিয়ে wrap করো।
4. try/except/else/finally-এর প্রতিটি block কোন condition-এ চলে, তা test করো।
5. Type hints-সহ CSV row parser লেখো।
6. `TypedDict` দিয়ে API payload-এর required ও optional field describe করো।
7. একই data model `TypedDict` এবং dataclass দিয়ে implement করে পার্থক্য লেখো।
8. Protocol ব্যবহার করে `close()` method থাকা object গ্রহণ করে এমন function লেখো।
9. Generic `first_item` function লেখো এবং mypy/pyright দিয়ে type inference পরীক্ষা করো।
10. Type hint ভুল হলেও runtime-এ কী ঘটে তা observe করো; তারপর runtime validation যোগ করো।

## Practice করার পদ্ধতি

1. Happy path-এর পাশাপাশি invalid, missing এবং boundary input test করো।
2. কোন exception recoverable, কোনটি propagate হওয়া উচিত তা লিখে রাখো।
3. Static checker চালিয়ে error বুঝে ঠিক করো; annotation-কে runtime validation ধরে নিও না।
4. Public function-এর input/output contract type hints-এ স্পষ্ট করো।
5. Error message-এ internal secret বা sensitive input প্রকাশ কোরো না।
